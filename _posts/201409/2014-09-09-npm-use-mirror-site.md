---
layout: post
title:  "npm 使用镜像网站"
date:   2014-09-09 09:40:00
categories: tool
tags: javascript
author: "Victor"
---

基于众所周知的原因，在国内使用npm下载资源的时候非常缓慢，甚至失败。使用一个靠谱的国内代理镜像是较好地解决方案。

### 命令行临时指定，可以临时指定 npm 镜像。

```bash
npm --registry https://registry.npm.taobao.org info underscore
```

### 通过config配置

```bash
npm config set registry https://registry.npm.taobao.org
```

### 编辑配置文件 ```~/.npmrc```

```bash
registry = https://registry.npm.taobao.org
```

### 一些国内的镜像

* http://r.cnpmjs.org/
* http://registry.npm.taobao.org
* http://npm.cbyun.com

其中淘宝的镜像可以用来代替官方版本(只读)，同步频率目前为 10分钟 一次以保证尽量与官方服务同步。

你可以使用淘宝定制的 cnpm (gzip 压缩支持) 命令行工具代替默认的 npm:

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

或者添加 npm 的 alias:

```bash
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"
# Or alias it in .bashrc or .zshrc
$ echo '\n#alias for cnpm\nalias cnpm="npm --registry=https://registry.npm.taobao.org \
  --cache=$HOME/.npm/.cache/cnpm \
  --disturl=https://npm.taobao.org/dist \
  --userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshrc
```

这样你就可以通过cnpm来管理模块了：

```bash
cnpm install [name]
```

### 相关链接

* [原文来自：npm使用镜像网站](http://colobu.com/2014/08/19/npm-local-mirror/#more)
* [Web开发利器:介绍一款快速开发套件 (Node, Grunt, Bower和Yeoman)](http://colobu.com/2014/08/29/modern-web-development-tools/)
