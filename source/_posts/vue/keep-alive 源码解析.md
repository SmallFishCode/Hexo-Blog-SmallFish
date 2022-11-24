---
title: keep-alive 源码解析
date: 2022/11/25 01:31:00
tags: Vue
categories: Vue
cover: /img/bg8.jpg
---

# 1. Vue 当中的 keep-alive 和 HTTP 中的 keep-alive

由于 keep-alive 在 vue 和 http 中都有出现，所以我们在此处将二者一起讨论一下。

### 1. HTTP 中的 keep-alive

> Keep-Alive 是一个通用消息头，允许消息发送者暗示连接的状态，还可以用来设置超时时长和最大请求数。(MDN)

-   语法: `Keep-Alive: parameters`

-   参数: parameters

    -   一系列用逗号隔开的参数，每一个参数由一个标识符和一个值构成，并使用等号 ('=') 隔开。下述标识符是可用的：

    -   timeout：指定了一个空闲连接需要保持打开状态的最小时长（以秒为单位）。需要注意的是，如果没有在传输层设置 keep-alive TCP message 的话，大于 TCP 层面的超时设置会被忽略。

    -   max：在连接关闭之前，在此连接可以发送的请求的最大值。在非管道连接中，除了 0 以外，这个值是被忽略的，因为需要在紧跟着的响应中发送新一次的请求。HTTP 管道连接则可以用它来限制管道的使用。

-   示例

```js
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Thu, 11 Aug 2016 15:23:13 GMT
Keep-Alive: timeout=5, max=1000
Last-Modified: Mon, 25 Jul 2016 04:32:39 GMT
Server: Apache

(body)
```

### 2. Vue 中的 keep-alive

> <KeepAlive> 是一个内置组件，它的功能是在多个组件间动态切换时缓存被移除的组件实例。（Vue 官网）

1. 是什么？

    - keep-alive 是一个 Vue 全局组件
    - keep-alive 本身不会渲染出来，也不会出现在父组件链中
    - keep-alive 包裹动态组件时，会缓存不活动的组件，而不是销毁它们

2. 怎么用？
   keep-alive 接收三个参数： - include：可传字符串、正则表达式、数组，名称匹配成功的组件会被缓存 - exclude：可传字符串、正则表达式、数组，名称匹配成功的组件不会被缓存 - max：可传数字，限制缓存组件的最大数量, 超过 max 则按照 `LRU算法` 进行置换

include 和 exclude，传数组情况居多.

-   基础用法:

```HTML
<!-- 非活跃的组件将会被缓存！ -->
<KeepAlive>
  <component :is="activeComponent" />
</KeepAlive>
```

-   动态组件

```js
<keep-alive :include="allowList" :exclude="noAllowList" :max="amount">
    <component :is="currentComponent"></component>
</keep-alive>
```

-   路由组件

```js
<keep-alive :include="allowList" :exclude="noAllowList" :max="amount">
    <router-view></router-view>
</keep-alive>
```

其他用法是本文重点, 会在下一小点中提到。

---

# 2. Vue 中 keep-alive 的进阶用法

### 1. props

```TS
interface KeepAliveProps {
  /**
   * 如果指定，则只有与 `include` 名称
   * 匹配的组件才会被缓存。
   */
  include?: MatchPattern
  /**
   * 任何名称与 `exclude`
   * 匹配的组件都不会被缓存。
   */
  exclude?: MatchPattern
  /**
   * 最多可以缓存多少组件实例。
   */
  max?: number | string
}

type MatchPattern = string | RegExp | (string | RegExp)[]
```

### 2. 详细使用

> <KeepAlive> 包裹动态组件时，会缓存不活跃的组件实例，而不是销毁它们。

任何时候都只能有一个活跃组件实例作为 <KeepAlive> 的直接子节点。

当一个组件在 <KeepAlive> 中被切换时，它的 activated 和 deactivated 生命周期钩子将被调用，
用来替代 mounted 和 unmounted。这适用于 <KeepAlive> 的直接子节点及其所有子孙节点。

### 3. include / exclude

```js
<!-- 用逗号分隔的字符串 -->
<KeepAlive include="a,b">
  <component :is="view"></component>
</KeepAlive>

<!-- 正则表达式 (使用 `v-bind`) -->
<KeepAlive :include="/a|b/">
  <component :is="view"></component>
</KeepAlive>

<!-- 数组 (使用 `v-bind`) -->
<KeepAlive :include="['a', 'b']">
  <component :is="view"></component>
</KeepAlive>
```

### 4. max

```js
<KeepAlive :max="10">
  <component :is="view"></component>
</KeepAlive>
```

### 5. 生命周期

```js
<script setup>
	import {(onActivated, onDeactivated)} from 'vue' onActivated(() =>{' '}
	{
		// 调用时机为首次挂载
		// 以及每次从缓存中被重新插入时
	}
	) onDeactivated(() => {
		// 在从 DOM 上移除、进入缓存
		// 以及组件卸载时调用
	})
</script>
```

### 6. 使用场景

可能大家在平时的开发中会经常遇到这样的场景：有一个可以进行筛选的列表页 List.vue，点击某一项时进入相应的详情页面，等到你从详情页返回 List.vue 时，发现列表页居然刷新了！刚刚的筛选条件都没了！！！

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c2bdfbc3aec43c388f414a9c74001ed~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

---

# 3. keep-alive 源码解析（V2）

### 1. keep-alive 在各个生命周期里都做了啥：

1. created：初始化一个 cache、keys，前者用来存缓存组件的虚拟 dom 集合，后者用来存缓存组件的 key 集合

2. mounted：实时监听 include、exclude 这两个的变化，并执行相应操作

3. destroyed：删除掉所有缓存相关的东西

### 2. 首先我们在 Vue2 源码中找到这个文件

```js
// src/core/components/keep-alive.js

export default {
	name: 'keep-alive',
	abstract: true, // 判断此组件是否需要在渲染成真实DOM
	props: {
		include: patternTypes,
		exclude: patternTypes,
		max: [String, Number],
	},
	created() {
		this.cache = Object.create(null) // 创建对象来存储  缓存虚拟dom
		this.keys = [] // 创建数组来存储  缓存key
	},
	mounted() {
		// 实时监听include、exclude的变动
		this.$watch('include', (val) => {
			pruneCache(this, (name) => matches(val, name))
		})
		this.$watch('exclude', (val) => {
			pruneCache(this, (name) => !matches(val, name))
		})
	},
	destroyed() {
		for (const key in this.cache) {
			// 删除所有的缓存
			pruneCacheEntry(this.cache, key, this.keys)
		}
	},
	render() {
		// 下面讲
	},
}
```

> keep-alive 不会被渲染到页面上，所以 abstract 这个属性至关重要！Vue2

我们可以看到在 destroyed 生命周期中，调用了 `pruneCacheEntry` 方法，那么我们找到这个函数定义来看看

### 3. pruneCacheEntry 函数

```js
// src/core/components/keep-alive.js

function pruneCacheEntry(cache: VNodeCache, key: string, keys: Array<string>, current?: VNode) {
	const cached = cache[key]
	if (cached && (!current || cached.tag !== current.tag)) {
		cached.componentInstance.$destroy() // 执行组件的destory钩子函数
	}
	cache[key] = null // 设为null
	remove(keys, key) // 删除对应的元素
}
```

总结一下就是做了三件事：

1. 遍历集合，执行所有缓存组件的$destroy 方法
2. 将 cache 对应 key 的内容设置为 null
3. 删除 keys 中对应的元素

### 4. render 函数

看完缓存组件的卸载，我们在看看 render 函数的作用

> 以下称 include 为白名单，exclude 为黑名单 render 函数里主要做了这些事：

1. 第一步：获取到 keep-alive 包裹的第一个组件以及它的组件名称

2. 第二步：判断此组件名称是否能被白名单、黑名单匹配，如果不能被白名单匹配 || 能被黑名单匹配，则直接返回 VNode ，不往下执行，如果不符合，则往下执行第三步

3. 第三步：根据组件 ID、tag 生成缓存 key，并在缓存集合中查找是否已缓存过此组件。如果已缓存过，直接取出缓存组件，并更新缓存 key 在 keys 中的位置（这是 LRU 算法的关键），如果没缓存过，则继续第四步

4. 第四步：分别在 cache、keys 中保存此组件以及他的缓存 key，并检查数量是否超过 max，超过则根据 LRU 算法进行删除

5. 第五步：将此组件实例的 keepAlive 属性设置为 true，这很重要哦，下面会讲到的！

```js
// src/core/components/keep-alive.js

render() {
  const slot = this.$slots.default
  const vnode: VNode = getFirstComponentChild(slot) // 找到第一个子组件对象
  const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions
  if (componentOptions) { // 存在组件参数
    // check pattern
    const name: ?string = getComponentName(componentOptions) // 组件名
    const { include, exclude } = this
    if ( // 条件匹配
      // not included
      (include && (!name || !matches(include, name))) ||
      // excluded
      (exclude && name && matches(exclude, name))
    ) {
      return vnode
    }

    const { cache, keys } = this
    const key: ?string = vnode.key == null // 定义组件的缓存key
      // same constructor may get registered as different local components
      // so cid alone is not enough (#3269)
      ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
      : vnode.key
    if (cache[key]) { // 已经缓存过该组件
      vnode.componentInstance = cache[key].componentInstance
      // make current key freshest
      remove(keys, key)
      keys.push(key) // 调整key排序
    } else {
      cache[key] = vnode // 缓存组件对象
      keys.push(key)
      // prune oldest entry
      if (this.max && keys.length > parseInt(this.max)) { // 超过缓存数限制，将第一个删除
        pruneCacheEntry(cache, keys[0], keys, this._vnode)
      }
    }

    vnode.data.keepAlive = true // 渲染和执行被包裹组件的钩子函数需要用到
  }
  return vnode || (slot && slot[0])
}
```

### 5. 如何渲染

咱们先来看看 Vue 一个组件是怎么渲染的，咱们从 render 开始说：

-   render：此函数会将组件转成 VNode

-   patch：此函数在初次渲染时会直接渲染根据拿到的 VNode 直接渲染成真实 DOM，第二次渲染开始就会拿 VNode 会跟旧 VNode 对比，打补丁（diff 算法对比发生在此阶段），然后渲染成真实 DOM

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/012505072dbb40e39c5edee63e11fc21~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 6. keep-alive 本身如何做到不渲染

刚刚说了，keep-alive 自身组件不会被渲染到页面上，那是怎么做到的呢？其实就是通过判断组件实例上的 `abstract` 的属性值，如果是 true 的话，就跳过该实例，该实例也不会出现在父级链上

```js
// src/core/instance/lifecycle.js

export function initLifecycle(vm: Component) {
	const options = vm.$options
	// 找到第一个非abstract的父组件实例
	let parent = options.parent
	if (parent && !options.abstract) {
		while (parent.$options.abstract && parent.$parent) {
			parent = parent.$parent
		}
		parent.$children.push(vm)
	}
	vm.$parent = parent
	// ...
}
```

### 7. 包裹组件渲染

咱们再来说说被 keep-alive 包裹着的组件是如何使用缓存的吧。刚刚说了 VNode -> 真实 DOM 是发生在 patch 的阶段，而其实这也是要细分的：VNode -> 实例化 -> \_update -> 真实 DOM，而组件使用缓存的判断就发生在实例化这个阶段，而这个阶段调用的是 `createComponent` 函数，那我们就来说说这个函数吧：

```js
// src/core/vdom/patch.js

function createComponent(vnode, insertedVnodeQueue, parentElm, refElm) {
	let i = vnode.data
	if (isDef(i)) {
		const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
		if (isDef((i = i.hook)) && isDef((i = i.init))) {
			i(vnode, false /* hydrating */)
		}

		if (isDef(vnode.componentInstance)) {
			initComponent(vnode, insertedVnodeQueue)
			insert(parentElm, vnode.elm, refElm) // 将缓存的DOM（vnode.elm）插入父元素中
			if (isTrue(isReactivated)) {
				reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
			}
			return true
		}
	}
}
```

-   在第一次加载被包裹组件时，因为 keep-alive 的 render 先于包裹组件加载之前执行，所以此时 vnode.componentInstance 的值是 undefined，而 keepAlive 是 true，则代码走到 i(vnode, false /_ hydrating _/)就不往下走了

-   再次访问包裹组件时，vnode.componentInstance 的值就是已经缓存的组件实例，那么会执行 insert(parentElm, vnode.elm, refElm)逻辑，这样就直接把上一次的 DOM 插入到了父元素中。

# 4. 收尾

在看 keep-alive 源码的过程中发现 Vue2 和 Vue3 当中部分地方还是有区别的，后续可能会出一期 Vue3 keep-alive 的文章。

另外，在本文提到的 LRU 算法，会单独更新一篇文章~
