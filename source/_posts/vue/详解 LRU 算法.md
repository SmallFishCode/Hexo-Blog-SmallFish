---
title: 详解 LRU 算法
date: 2022/11/25 19:58:00
tags: Vue
categories: Vue
cover: /img/bg8.jpg
---

# 1. 什么是 LRU 算法？

> LRU (Least Recently Used 最近最少使用)

> LRU用通俗的话来说就是最近被频繁访问的数据会具备更高的留存，淘汰那些不常被访问的数据。

力扣中有一道 LRU 算法题大家可以看看：https://leetcode.cn/problems/lru-cache/
可以先理解题目意思，如果不理解我们接着往下走

# 2. 我们用几张图来理解一下

假设我是一个卖玩具的商人，我在街上租了一个只能放下三个玩具（没办法太穷了）的摊位，所以大部分的玩具都没摆出来而是放在仓库里。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6332e1e3e03f43948bfa7cc2c621ace2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

好家伙，终于有个人来问了，我赶紧把一个玩具摆在了第一个格子...

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24067c0e5af04ebabae198e32410bc0e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

生意太好了，又有人要问自行车，因为第一个格子是黄金位置，所以我把最新的都放那...

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72139a3a166a4820b929ab8a4d5ccba3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

太火爆了，马上我的三个格子就不够用了...

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ef83afb2ce04aad92f595b5bc55c425~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

因为格子从上到下依次最受欢迎，所以我会把下面格子的玩具放回到仓库，给新的玩具腾出点地方来

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a1cdf3450044a3ba6464cac5edcf2ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

当然啦，如果客户想看摊位上已经有的玩具，我会把他放到第一个最火热的格子

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab9c9324ab34410291df273f85dfdec5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

# 3. 理解 LRU

1. 有一个最大缓存数量 capacity (是一个正整数) 在上图中代表容量
2. 有一个 cache 作为容器
3. 当有人要看某一个玩具时，就是 get 方法，取出对应的玩具，并更新它的位置，以便我们下架最冷清的玩具和取出最火爆的玩具。
4. 当我们要上架一个玩具时，先判断存不存在货架上，如果不存在，则直接加在最后面，如果存在，就刷新它的位置，放到最前面。

# 4. LRU 的应用场景

1. Vue 的全局组件 keep-alive 的缓存原理
2. redis 等缓存排序
...

# 5. LRU 算法实现

### 1. 数组方式实现

```js
// LRU 数组实现

let arr = ['a', 'b', 'c', 'd'] // 'e' // 找出哪个最久没被访问, 用 'e' 替换掉
// 访问完，放到最前面或最后面

let LRUCache = function (capacity) {
	this.capacity = capacity
	this.cache = []
}

// 拿到访问的数据，并刷新它在缓存集合当中的位置
LRUCache.prototype.get = function (key) {
	let index = this.cache.findIndex((item) => item.key)
	if (index === -1) {
		return -1
	}
	let { value } = this.cache[index]
	this.cache.splice(index, 1) // 移除访问的元素
	this.cache.unshift({
		key,
		value,
	})
	return value
}

// 新增一个访问者
LRUCache.prototype.put = function (key, value) {
	let index = this.cache.findIndex((item) => item.key === key) // 判断是否存在
	if (index > -1) {
		// 如果存在，则把它刷新位置
		this.cache.splice(index, 1)
	} else if (this.cache.length >= this.capacity) {
		// 没找到, 并且访问者数量已经达到最大
		this.cache.pop()
	}
	this.cache.unshift({
		key,
		value,
	})
}
```

- 其实我们对着注释理解起来，感觉不难，但是我们注意到，在数组的这种实现方式中，我们用到了很多次的 `findIndex` `splice` `unshift` 等方法，那么自然时间复杂度都在 O(n) 之上，所以有没有时间复杂度为 O(1) 的方法呢？

- 我们仔细想想有哪种数据结构查询某个值和取用某个值是 O(1) ? 没错！ 它就是 哈希！！

### 2. 哈希实现

```js
let LRUCache = function (capacity) {
	this.capacity = capacity
	this.cache = new Map()
}

// 拿到访问的数据，并刷新它在缓存集合当中的位置
// map 的 set 一定会放到最后面
// map 取值内部很复杂
LRUCache.prototype.get = function (key) {
	if (this.cache.has(key)) {
		let temp = this.cache.get(key)
		// 更新位置
		this.cache.delete(key)
		this.cache.set(key, temp)
		return temp
	}
	return -1
}

// 新增一个访问者
LRUCache.prototype.put = function (key, value) {
	if (this.cache.has(key)) {
		// 更新位置
		this.cache.delete(key)
	} else if (this.cache.size >= this.capacity) {
		// 移除掉最久没访问的, 如何找到？
		// map.keys() 返回一个 key 的迭代器对象，使用 next 方法往下执行，拿到第一个值
		// this.cache.keys().next().value
		this.cache.delete(this.cache.keys().next().value)
	}
	this.cache.set(key, value)
}
```

- 哈希实现的几个注意点：
    - `this.cache.has(key)` 判断是否存在
    - `this.cache.set(key, temp)` 将值存入 hash
    - `this.cache.delete(key)` 删除该 hash 表中的 key
    - `this.cache.size` 获取到 map 的长度 
    - `this.cache.keys()` 获取到一个 迭代器对象里面存放 keys ，通过 next 方法获取第一个 value
    - map 的特性：
    
    ```js
    let a = new Map()
    undefined
    a.set(1, 1)
    a.set(2, 2)
    a.set(3, 3)
    Map(3) {1 => 1, 2 => 2, 3 => 3}
    a.delete(2)
    true
    a
    Map(2) {1 => 1, 3 => 3}
    a.set(4,4)
    Map(3) {1 => 1, 3 => 3, 4 => 4}
    ```
    
# 6. 收尾

LRU 算法可以结合 Vue 中 keep-alive 源码分析~ 入口：[Vue keep-alive 源码解析](http://smallfish.space/2022/11/25/vue/keep-alive%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)