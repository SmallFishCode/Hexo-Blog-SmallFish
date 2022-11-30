---
title: Fish-Node 使用文档
date: 2022/11/30 13:59:45
tags: Node
categories: Node
cover: /img/bg9.jpg
---

# 1. 目录配置

目前支持:
- config: 配置数据库信息
- controller: 业务逻辑存放
- model: 数据库相关配置
- routes: 路由文件存放
- service: 服务层文件
- index.js: 入口文件

# 2. Index.js

### 1. 使用规则 (约定)：

- npm i
- 引入 `fish-node`，并创建实例
- 调用 go 方法启动在 (任意) 3000 端口

```js
const Fish = require('fish-node')
const fish = new Fish()

fish.go(3000)
```

# 3. Routes

### 1. 路由文件

- 入口文件: index.js

### 2. 使用规则:

- 仅需在 `module.exports = {}` 中抛出对应的业务逻辑

- 一个接口对应：method + path + 业务逻辑 (业务逻辑抽离到 controller 文件夹)

```js
module.exports = (fish) => ({
	'get /': fish.$ctrl.home.index,
	'get /detail': fish.$ctrl.home.detail,
	'get /test': fish.$ctrl.home.test,
})
```

- 多个路由文件，直接创建文件即可，不用导入导出，自动引入，例如，在 routes 下创建 user.js

- 约定语法和上面 index.js 中一样，直接抛出，此处并没有将业务逻辑抽离到 controller 中，看个人习惯。

- 注意到此处的 ctx 并不是直接使用，而是需要使用 fish 调用 ctx

```js
module.exports = {
	'get /': async (fish) => {
		const name = await fish.$service.user.getName()
		fish.ctx.body = '用户名：' + name
	},
	'get /detail': async (fish) => {
		const age = await fish.$service.user.getAge()
		fish.ctx.body = '用户年龄：' + age
	},
}
```

# 4. Contorller

> 业务逻辑层

### 1. 使用规则：

- 与整体规则相似，直接抛出

```js
module.exports = {
	index: async (fish) => {
		fish.ctx.body = 'Ctrl Home'
	},
	detail: async (fish) => {
		fish.ctx.body = 'Ctrl Detail'
	},
	test: async (fish) => {
		fish.ctx.body = 'Test fish-node！！'
	},
}
```

- 注意到，此处键名是对应的路径，例如我要在用户访问 `/detail` 时返回 Ctrl Detail ，那我只需要在 routes 中使用 fish 关键字：

- fish.$ctrl.home.detail home 指的是你在 controller 目录下对应文件的名字，detail 则是抛出对象的键名

```js
module.exports = (fish) => ({
	'get /detail': fish.$ctrl.home.detail,
})
```

# 5. MySQL 接入

> 目前只支持 MySQL

### 1. 使用规则

- 只需要在 config 的 index.js 中添加如下配置即可

```js
module.exports = {
	db: {
		dialect: 'mysql', // 使用的数据库名
		host: 'localhost', // 地址
		database: 'fish', // 库名
		username: 'root', // 用户名
		password: '******', // 密码
	},
}
```

- 例如此时我需要访问 localhost:3000 的时候从数据库中读取数据，只需要在 controller 对应文件下：

```js
module.exports = {
	index: async (fish) => {
		fish.ctx.body = await fish.$model.user.findOne({ where: { id: 2 } })
	},
}
```

- 此处使用的 mysql2 的搜索语法


# 6. 持续更新中...