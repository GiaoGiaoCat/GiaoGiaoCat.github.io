---
layout: post
title:  "Brew 管理 go 版本"
date:   2019-10-07 17:10:00

categories: go
tags: tip
author: "Victor"
---

## 起因

接手了一个项目，其中一个 package 不支持 1.13 只好回退到 1.12 去。

## 步骤

### 查看可安装版本

先更新 brew 然后查看可安装的 golang 版本。

```bash
brew update
brew search go
```

```bash
➜ brew search go
==> Formulae
algol68g                         go-statik                        gollum                           gost                             mongo-orchestration
anycable-go                      go@1.10                          golo                             gosu                             mongoose
arangodb                         go@1.11                          gom                              gotags                           pango
argon2                           go@1.12                          gomplate                         goto                             pangomm
aws-google-auth                  go@1.9                           goocanvas                        gource                           protoc-gen-go
```

可以看到有 1.12 版本。

### 安装

因为已经安装过 1.13 版本了，现在装 1.12 版本。

```bash
brew install go@1.12
```

查看所有版本。

```bash
# way 1
brew info go

# way 2
brew ls --versions | grep go
```

这时候你会发现，我们安装的 go 1.12 放在了一个特殊的目录 `local/Cellar/go@1.12` 下，而不是放在 `local/Cellar/go` 目录下。其它版本的规则查看 https://formulae.brew.sh/formula/go 。

那我们需要把目录自己移动过去。

```bash
cp -R /usr/local/Cellar/go@1.12/1.12.10 /usr/local/Cellar/go/
```

现在再查看版本 `brew info go` 就正确了。

### 切换版本

```bash
brew switch go 1.12.10
```

屏幕显示。

```bash
Cleaning /usr/local/Cellar/go/1.12.10
Cleaning /usr/local/Cellar/go/1.13
3 links created for /usr/local/Cellar/go/1.12.10
```

## 参考

* [mac上切換多個go版本](https://babygoat.github.io/2019/06/19/Golang-mac%E4%B8%8A%E5%88%87%E6%8F%9B%E5%A4%9A%E5%80%8Bgo%E7%89%88%E6%9C%AC/)
* [透過brew來做Golang的版本切換](https://www.evanlin.com/go-version-control-with-brew/)
