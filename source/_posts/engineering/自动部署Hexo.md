---
title: 自动部署
date: 2022/11/22 0:10:00
tags: 前端 工程化 自动部署
categories: 工程化
cover: /img/6.jpg
---

# 前言

    Hello，我是小鱼。

    这个博客是我 2022 年春节时期搭建并部署在阿里云 ECS 服务器上的，后续由于 hexo 静态博客每次更改完文件之后，就得从新 hexo c g d。

    所以时隔多日， 我决定改造一番，本次改造主要是通过 github actions 进行 CI/CD，免去了手动清理，打包，推送到服务器的繁琐流程。

    在后面我会介绍到整个改造的流程以及关于 github actions 的简单介绍和用法讲解~

### 什么是 CI/CD ？
