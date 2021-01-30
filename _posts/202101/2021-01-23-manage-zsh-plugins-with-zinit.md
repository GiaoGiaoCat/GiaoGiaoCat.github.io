---
layout: post
title:  "使用 Zinit 管理 Zsh Plugins"
date:   2021-01-23 12:00:00

categories: tool
tags: terminal
author: "Victor"
---

## 简介

之前用 antigen 来管理插件，觉得加载有点慢呢，新年来折腾下 zinit 了。

### 安装

```bash
git clone https://github.com/zdharma/zinit.git /usr/local/share/zinit/bin
```

然后在你的 ~/.zshrc 顶端添加如下语句

```bash
source /usr/local/share/zinit/bin/zinit.zsh
```

## 相关阅读
