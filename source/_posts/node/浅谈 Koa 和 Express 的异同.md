---
title: 浅谈 Koa 与 Express 的异同
date: 2022/11/24 21:52:45
tags: Node
categories: Node
cover: /img/bg9.jpg
---

# 1. 两者的联系

1. Express 和 Koa 都是出自 [TJ Holowaychuk](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftj) 大神之手。
 
2. Express 是一个基于 Node.js 平台的 web 应用开发框架。而 Koa 则是 Express 的 `升级版` Koa 诞生之初正值 Nodejs 推出 `async/await` 语法之时， Koa 采用这种新的语法特性，丢弃回调函数，实现了了一个轻量优雅的 web 后端框架。

3. 两者都采用中间件方式进行开发，并且相关 api 基本是大同小异的。

---

# 2. 二者的区别

### 1. 架构设计

Express是一个集合式的框架，自身集成了router，static，views，bodyparse等等常用中间件。这样的设计让开发者能非常快速的搭建一个node后端服务，这个服务功能齐全。可以实现接口服务，静态资源服务，页面服务，甚至文件上传也能实现。但是使用Express就必须引入全部的中间件和功能，不管你有没有用到。

Koa2的设计思想就是小而美，轻量，插件化设计。只提供最基础的框架，所有功能都通过中间件引入。Koa将Express集成的中间件进行拆分，开发者按需引入即可使用。比如：

得益于这种设计，用户在选择中间件时有了更多的选项。你可以选择koa提供的中间件(如果koa提供的话)，也可以选择社区或第三方开发的中间件，甚至自己写一个都可以。这在Express上是无法想象的。

Koa 是 Express 原班人马基于 `ES6` 新特性重新开发的框架，主要基于 co 中间件，框架自身不包含任何中间件，很多功能需要借助第三方中间件解决，但是由于其基于 ES6 generator 特性的异步流程控制，解决了 "callback hell" 和麻烦的错误处理问题。

总结来说，Express是一个功能齐全，开箱即用的集合式框架，比较重。Koa2是一个插件化的轻量框架，功能按需引入。

### 2. 中间件模型

##### N1: 中间件模型结构差异

Express 的中间件模型为 `线型`，而 Koa 的中间件模型为 U 型，也可称为 `洋葱模型` 构造中间件

- Express 线型模型示例：
```js
const express = require("express");
const app = express();
const port = 3000;

app.use((req, res, next) => {
  res.write("hello");
  next();
});
app.use((req, res, next) => {
  res.set("Content-Type", "text/html;charset=utf-8");
  next();
});
app.use((req, res, next) => {
  res.write("world");
  res.end();
});
app.listen(3000);
```

Express 中间件有趣的地方：
```js
const Express = require('express')
const app = new Express();
const sleep = () => new Promise(resolve => setTimeout(function(){resolve(1)}, 2000))
const port = 3000

function f1(req, res, next) {
  console.log('f1 start ->');
  next();
  console.log('f1 end <-');
}

function f2(req, res, next) {
  console.log('f2 start ->');
  next();
  console.log('f2 end <-');
}

async function f3(req, res) {
  //await sleep();
  console.log('f3 service...');
  res.send('Hello World!')
}

app.use(f1);
app.use(f2);
app.use(f3);
app.get('/', f3)
app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

控制台执行 curl localhost:3000 输出如下，有点迷惑了，不是线性吗？为什么和我们上面讲 Koa 的输出顺序是一致呢？不也是洋葱模型吗？
```js
f1 start ->
f2 start ->
f3 service...
f2 end <-
f1 end <-
```

少年，先莫及，再看一段代码。
上面我们的 f3 函数其中注释了一条代码 await sleep() 延迟执行，现在让我们打开这个注释。

```js
async function f3(req, res) {
  await sleep(); // 改变之处
  console.log('f3 service...');
  res.send('Hello World!')
}
```

控制台再次执行 curl localhost:3000，发现顺序发生了改变，上游中间件并没有等待 f3 函数执行完毕，就直接执行了。

```js
f1 start ->
f2 start ->
f2 end <-
f1 end <-
f3 service...
```

> Express 中间件实现是基于 Callback 回调函数同步的，它不会去等待异步（Promise）完成，这也解释了为什么上面的 Demo 我加上异步操作，顺序就被改变了。
在 Koa 的中间件机制中使用 Async/Await（背后全是 Promise）以同步的方式来管理异步代码，它则可以等待异步操作。

- Koa 洋葱模型示例：
```js
import * as Koa from "koa";
const app = new Koa();
app.use(async (ctx, next) => {
  ctx.body = "hello";
  await next();
  ctx.body += "world";
});
app.use(async (ctx) => {
  ctx.set("Content-Type", "text/html;charset=utf-8");
});
app.listen(3000);
```

可以看到，Express 实现同样的功能，要比 Koa 多一步。

##### N2: 洋葱模型实际应用场景

- 使用 node 服务监控一个请求的响应时间
```js
// 中间件源码部分
module.exports = (options) => {
    const {koaRouterPathAggregate, ...baseOptions } = options;
    
    const metrics = new BaseServiceMetrics(baseOptions);
    const { entry, exit } = metrics;
    return async (ctx, next) => {
        const linkObj = entry(ctx);
        await next();
        try {
            let otherObj = {};
            if (koaRouterPathAggregate) {
                const realPath = getKoaRouterPath(ctx, koaRouterPathAggregate);
                if (realPath) {
                  otherObj.path = realPath;
                }
            }
            exit(ctx, linkObj, otherObj);
        } catch(ex) {
            console.log(ex);
        }
    }
}

// 服务使用部分
var http = require('http');
http.createServer(function(req, res){
 const entryObj = entry(req); // 开始监控统计入口，一般在请求开始时执行
 /* todo 计算业务逻辑
 res.writeHead(200, {'Content-type' : 'text/html'});
 res.write('<h1>Node.js</h1>');
 res.end('<p>Hello World</p>');
 */
 exit(res, entryObj); // 开始监控统计出口，一般在计算完执行准备返回时执行
}).listen(3000);
```

业务逻辑开始前调用 entry，逻辑结束后 调用 exit，内部通过 node 自带的 `process.hrtime()` 获取到当前时间。

- 需要回调下一个中间件的处理结果 (待补充)


如果 Express 也想实现 Koa 的洋葱模型就比较困难，需要开发者在每一个中间件上都添加回调函数。因此 Express 项目真的很难直接使用 Koa 的中间件机制，怪不得 TJ Holowaychuk 大神也要另起炉灶。

### 3. 异步方式

- Express 通过回调实现异步函数，在多个回调、多个中间件中写起来容易逻辑混乱。
```js
// express写法
app.get("/test", function (req, res) {
  fs.readFile("/file1", function (err, data) {
    if (err) {
      res.status(500).send("read file1 error");
    }
    fs.readFile("/file2", function (err, data) {
      if (err) {
        res.status(500).send("read file2 error");
      }
      res.type("text/plain");
      res.send(data);
    });
  });
});
```

- Koa 通过 generator 和 async/await 使用同步的写法来处理异步，明显好于 callback 和 promise。

```js
app.use(async (ctx, next) => {
  await next();
  var data = await doReadFile();
  ctx.response.type = "text/plain";
  ctx.response.body = data;
});
```

### 4. 捕获错误
- Express 沿用 Node.js 的 Error-First 的模式（第一个参数是 error 对象）来捕获错误。由于Express的中间件时线性流程，所以要处理错误信息就必须把error中间件放到最后。
```js
app.use(function (err, req, res, next) {
  console.error(err.stack);
  res.status(500).send("Something broke!");
});
```

- Koa 使用 try/catch 的方式来捕获错误。
```js
app.use(async (ctx, next) => {
  try {
    await next();
  } catch (error) {
    if (error.errorCode) {
      console.log("捕获到异常");
      return (ctx.body = errror.msg);
    }
  }
});
```

### 5. 响应机制

- Express 立刻响应（res.json/res.send），上层不能再定义其他处理。
- Koa 中间件执行完之后才响应（ctx.body = xxx），每一层都可以对响应进行自己的处理

