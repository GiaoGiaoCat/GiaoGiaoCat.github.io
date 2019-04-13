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
export PATH="$PATH:$(go env GOPATH)/bin:$(go env GOROOT)/bin:$GOPATH/bin"
```

```bash
$ go env
$ go version
```

### 环境变量

* $GOROOT 表示 Go 在你的电脑上的安装位置，它的值一般都是 `$HOME/go`，你也可以安装在别的地方。
* $GOPATH 默认采用和 $GOROOT 一样的值，但从 Go 1.1 版本开始，你必须修改为其它路径。

**Go 编译器支持交叉编译，也就是说你可以在一台机器上构建运行在具有不同操作系统和处理器架构上运行的应用程序，也就是说编写源代码的机器可以和目标机器有完全不同的特性（操作系统与处理器架构）。**

为了区分本地机器和目标机器，你可以使用 `$GOHOSTOS` 和 `$GOHOSTARCH` 设置目标机器的参数，这两个变量只有在进行交叉编译的时候才会用到。

### 格式化代码

建议在提交代码前统一格式化代码 `gofmt -w *.go`。 `gofmt map1` 会格式化并重写 map1 目录及其子目录下的所有 Go 源文件。

如果不加参数 -w 则只会打印格式化后的结果而不重写文件。

### 生成代码文档

`go doc` 工具会从 Go 程序和包文件中提取顶级声明的首行注释以及每个对象的相关注释，并生成相关文档。

详见这里 https://www.kancloud.cn/kancloud/the-way-to-go/72451

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

### Debug

Go 的 debug 工具首推 delve，先学习一下如何使用。

#### 命令行下使用

首先就是安装：

1. `xcode-select --install`
2. `go get -u github.com/go-delve/delve/cmd/dlv`

用法也不算难：

1. 假如你的项目在 `src/github.com/me/foo/cmd/foo` 目录下，并且你就在这个目录，可以直接执行 `dlv debug` 进入 debug 命令行模式。
2. 如果在其它目录，可以 `dlv debug github.com/me/foo/cmd/foo` 用来调试该目录下 `cmd/foo.go` 文件。
3. 然后用断点调试即可，比如想在 main 函数上价格断点就 `break main.main`

具体的详情直接看[Getting Started](https://github.com/go-delve/delve/blob/master/Documentation/cli/getting_started.md)

#### VSCode 配合 delve

首先安装官方的 VSCode Go 扩展，然后配置好 `GOPATH`。

1. 这一步不能忽略 `go get -u github.com/go-delve/delve/cmd/dlv`
2. 在 VSCode command `shift + cmd + p` 中执行 Go: Install/Update Tools 可能需要重启 VSCode 才行。
3. 修改 .vscode/launch.json 文件
4. 打断点，在 debug 那里运行

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "remotePath": "",
            "port": 2345,
            "host": "127.0.0.1",
            "program": "${fileDirname}",
            "env": {},
            "args": [],
            "showLog": true
        }
    ]
}
```


* [Debugging Go with VS Code and Delve](https://flaviocopes.com/go-debugging-vscode-delve/)
* [Debugging Go Code with Visual Studio Code](https://scotch.io/tutorials/debugging-go-code-with-visual-studio-code)
* https://74th.github.io/vscode-debug-specs/golang/
* https://flaviocopes.com/go-debugging-vscode-delve/
* https://scotch.io/tutorials/debugging-go-code-with-visual-studio-code

### 作废

谢谢你看到这里，学习 Go 3 周之后，发现 1.11 开始更推荐使用 module 而不是 GOPATH，所以我会单独起一篇如何用 VSCode 开发 Go 的文章，本文关于编辑器部分的知识都过时了。

## 相关

* [Get Setup](http://www.golangbootcamp.com/book/get_setup)
* [Golang setup in Mac OSX with HomeBrew](https://gist.github.com/vsouza/77e6b20520d07652ed7d)
* [SettingGOPATH](https://github.com/golang/go/wiki/SettingGOPATH)
