---
title: PicGo + Github + VScode 定制图床
date: 2022/12/04 00:40:00
tags: Tools
categories: Tools
cover: /img/bg10.jpg
---

# 1. 为什么需要定制图床？

> 当我们在写博客的时候，经常会需要上传许多网络图片，试想一下，当图片来源被删除后，在你的博客中，只剩下了一张图片已失效的 `error`

### 1. 什么是图床？

-   图床好比你将一段文章 copy 到了本地的备忘录中，可以称为 `备份`

### 2. 解决了啥问题?

-   我们看到上面的问题，那么这个时候，如果我们有个图床，就相当于对这张图片进行了一次备份，存放在你指定的位置，不会受到源图片地址的干扰，岂不美哉！！

# 2. VScode + PicGo

-   打开 VScode 插件商店搜索 PicGo，并安装

-   打开 VScode 工作区设置，我们此处选择 `github` 托管服务

-   我们需要配置以下几个选项：

![2022-12-08-18-03-04_PicGo_Github_VScode](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-08-18-03-04_PicGo_Github_VScode)

-   此处自定义域名可以使用如下的链接，国内访问更快！

```js
https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo

https://cdn.jsdelivr.net/gh/用户名/仓库名
```

# 3. Github

-   我们打开 github 新建一个仓库，将信息填入到第二步当中的配置中

-   获取 token，打开 settings，看到如下：

![2022-12-08-18-02-21_PicGo_Github_VScode](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-08-18-02-21_PicGo_Github_VScode)

-   此时，点击右侧新建 token，设置过期时间，选中第一个：

![2022-12-08-18-02-51_PicGo_Github_VScode](https://cdn.jsdelivr.net/gh/SmallFishCode/PicGo/2022-12-08-18-02-51_PicGo_Github_VScode)

-   最后将得到的 token 填入到第二步配置中就好啦！
