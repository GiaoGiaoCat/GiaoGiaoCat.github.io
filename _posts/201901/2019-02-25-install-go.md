---
layout: post
title:  "安装和配置 Go"
date:   2019-02-25 13:00:00

categories: go
tags: editor
author: "Victor"
---

## Install Go Lang on MacOS with Homebrew!

```bash
# or install Go to be able to crosscompile
# brew install go --cross-compile-common
brew install go
```

默认的 GOPATH 是 `$HOME/go` 可以通过修改 .zshrc 的方式更改目录

```bash
export GOPATH=$HOME/Documents/Go
```

为了在终端的任何路径，都可识别 go 命令和 go 相关工具还可以增加如下配置：

```bash
# Edit your ~/.zshrc or ~/.bash_profile file to add the following line
export PATH="$PATH:$(go env GOPATH)/bin:$(go env GOROOT)/bin"
```

```bash
$ go env
$ go version
```

## 配置和使用 GoLand

在 Gogland 中，需要配置当前项目的 GOROOT，用来编译运行 Go 代码。首先在 Settings -> Go -> GOROOT 设置即可。

GOPATH 可用于 Go 导入、安装、构建和更新，会被 Gogland 自动识别。Gogland 中的 GOPATH 设置功能非常实用和强大，你既可以配置多个全局的 GOPATH （IDE 会自动识别环境变量中的 GOPATH，可不勾选），也可以配置多个项目级别的 GOPATH，甚至还可以配置多个模块级别的 GOPATH。

新建的项目会自动关联全局 GOPATH，你还可以设置项目的 GOPATH。

### 运行/调试/测试程序

为了在 Gogland 运行一个 Go 程序，你需要用到 Run Configuration。使用方法如下：

1. 在主菜单栏或工具栏打开：Run -> Edit Configurations
2. 点击 Edit Configurations，打开 Run/Debug Configuration 对话框
3. 点击 + 号按钮，选择你需要的运行配置。Go 用到的配置类型如下，前三项，最为常用：
  * Go Application：相当于执行 go build 和运行可执行文件命令，该配置会生成可执行文件，也可执行 debug
  * Go Single File：相当于 go run 命令，该配置不会生成可执行文件，不能执行 debug
  * Go Test：用于运行测试代码，相当于 go test，有三种测试框架可供选择：gotest，gocheck 和 gobench
  * Go Remote：提供了 Go 的远程调试支持，你只需要设置要远程连接的 Host 和 Port，并且保证你要调试的程序是通过 Delve 启动的
  * Go App Engine：允许你将程序部署到 Google AppEngine，前提是你有使用 Google 云，并且你的程序模块加载了 Go AppEngine SDK

如果你要调试程序，本地调试可用 Go Application，远程调试请使用 Go Remote。测试程序，请使用第 Go Test 种方式。

## VS Code

初学者的我使用 VSCode，而且也按照 [How to Write Go Code](https://golang.org/doc/code.html) 一文用最基础和简单的方式来组织 Go 的代码。

```bash
$ mkdir -p $GOPATH/src/github.com/user
$ mkdir $GOPATH/src/github.com/user/hello
```

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```

Build & Run：

```bash
# as same as `cd $GOPATH/src/github.com/user/hello & go install`
go install github.com/user/hello
```

Run the program by typing its full path at the command line：

```bash
$GOPATH/bin/hello
# as you have added $GOPATH/bin to your PATH, just type the binary name
hello
```

## 相关

* [Get Setup](http://www.golangbootcamp.com/book/get_setup)
* [Golang setup in Mac OSX with HomeBrew](https://gist.github.com/vsouza/77e6b20520d07652ed7d)
* [SettingGOPATH](https://github.com/golang/go/wiki/SettingGOPATH)
