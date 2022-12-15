---
title: 如何给开源项目提 pr ？
date: 2022/12/15 14:55:00
tags: 工程化 开源
categories: 工程化
cover: /img/bg3.jpg
---

# 1. 前言

> 在技术开发的过程中，我们时常会使用到很多开源的框架和库，那么我们能不能、为开源仓库贡献，成为贡献者呢？ 当然可以，本文我们来详细的讲解一下如何给一个开源项目提 pr ？

# 2. 流程

1. 首先我们在 github 中找到开源项目的仓库地址，并点击右上角的 fork 到本地仓库。

![2022-12-15-14-44-17_如何给开源项目提pr](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-15-14-44-17_如何给开源项目提pr)

2. 接着，我们回到本地仓库使用 git clone 克隆该仓库（使用 https 的方式）

3. 我们在本地修改代码，然后推到远程

```js
git add .
git commit -m 'fix: xxx'
git push origin xxx
```

4. 此时回到我们 github 的仓库，点击 pr，再点击 create pull request

![2022-12-15-14-49-13_如何给开源项目提pr](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-15-14-49-13_如何给开源项目提pr)

5. 填入 pr 信息，此时要注意，看看该开源项目有没有自己的提交规范，按照规范填写信息，最后点击创建

![2022-12-15-14-50-54_如何给开源项目提pr](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-15-14-50-54_如何给开源项目提pr)

6. 此时你的 pr 就出现在了开源项目中，只需要作者同意合并就完成啦！！

![2022-12-15-14-53-04_如何给开源项目提pr](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-15-14-53-04_如何给开源项目提pr)

# 3.总结

希望本文能帮助到各位想参与开源社区建设的小伙伴们 ~