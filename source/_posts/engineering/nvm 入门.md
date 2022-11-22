---
title: Nvm 入门
date: 2022/11/22 13:31:00
tags: 工程化 
categories: 工程化
cover: /img/6.jpg
---

# 11.21
https://github.com/nvm-sh/nvm

### 1. 安装
```js
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
```

### 2. 配置 
根据终端不同打开不同的配置文件：~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc
此处使用的是 zsh，使用 `code ~/.zshrc` 打开 zshrc

- bash: source ~/.bashrc
- zsh: source ~/.zshrc
- ksh: . ~/.profile


```js
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

### 3. 终端翻墙
代理到你翻墙软件的端口 此处是 33210
```js
export https_proxy=http://127.0.0.1:33210 http_proxy=http://127.0.0.1:33210 all_proxy=socks5://127.0.0.1:33210
```

### 4. 使用
```js
nvm install node v16.16.0    --- 安装 16.16.0 版本的 node
nvm ls-remote       --- 查看可使用的 node 版本
nvm use v12.0.0     --- 使用 12.0.0 版本的 node
```