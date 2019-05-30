---
layout: post
title:  "Go 项目的组织结构"
date:   2019-04-14 13:00:00

categories: go
tags: tip
author: "Victor"
---

## Web 项目

从 Rails 框架转过来搞 Go，所以首先想到的就是以 MVC 的价格方式来组织代码。

### 方案 1

1. main.go 作为应用入口，初始化一些运行博客所需要的基本资源，配置信息，监听端口。
2. 路由功能检查HTTP请求，根据 URL 以及 method 来确定谁(控制层)来处理请求的转发资源。
3. 如果缓存文件存在，它将绕过通常的流程执行，被直接发送给浏览器。
4. 安全检测：应用程序控制器调用之前，HTTP 请求和任一用户提交的数据将被过滤。
5. 控制器装载模型、核心库、辅助函数，以及任何处理特定请求所需的其它资源，控制器主要负责处理业务逻辑。
6. 输出视图层中渲染好的即将发送到Web浏览器中的内容。如果开启缓存，视图首先被缓存，将用于以后的常规请求。

```bash
|——main.go         # 入口文件
|——conf            # 配置文件和处理模块
|——controllers     # 控制器入口
|——models          # 数据库处理模块
|——utils           # 辅助函数库
|——static          # 静态文件目录
|——views           # 视图库
```

### 方案 2

```bash
projectName         # 项目名称
|-- app             # 应用程序目录
|---- controllers   # 控制器，入参校验
|---- middleware    # 中间件
|---- routers       # 路由
|---- services      # 业务逻辑处理
|-- config          # 配置文件
|-- dao             # 数据库访问
|-- models          # 数据模型
|-- storage         # 存储
|---- cache         # 缓存
|---- logs          # 日志
|-- main.go         # 程序入口
```

## 社区建议

实际上每个人都有每个人的风格，项目在不断地变大，一些新的文件或目录又不断地被添加进来，从这里面去找到自己需要的信息的成本越来越高，一个社区认可的统一的通用的目录结构非常有必要。

https://github.com/golang-standards/project-layout 是社区建议的方案。

* api - OpenAPI/Swagger specs, JSON schema files, protocol definition files 协议文件
* assets - 静态资源，比如图片、Logo 等
* build - 包和持续集成目录，Docker 配置和脚本放到 `/build/package/` 目录，CI 的脚本放在 `/build/ci`
* cmd - main 函数文件，比如 `/cmd/myapp.go` 这里的每个文件在编译之后都会生成一个可执行的文件。
* configs - 配置文件模板或默认配置
* deployments - 部署相关的配置文件和模板，比如：docker-compose, kubernetes/helm, mesos, terraform, bosh
* docs - 需求、设计文档，这里不要放能自动生成的文档
* examples - 应用程序或者公共库使用的一些例子
* githooks
* init - 服务启停脚本
* internal - 私有应用程序和私有库代码，
* pkg - 一些通用的可以被其他项目所使用的代码，放到这个目录下面
* scripts - 其他一些脚本，编译、安装、测试、分析等等
* test - 黑盒测试、冒烟测试、功能测试、性能测试以及测试需要的数据等。注意，这里放的不是 Go 的单元测试，这里的代码可能是 Ruby 或者 Python。
* third_party - External helper tools, forked code and other 3rd party utilities，比如 Swagger UI。 *可以忽略*
* tools - 常用的工具和脚本，可以引用 `/internal` 或者 `/pkg` 里面的库
* vendor - 项目依赖的其他第三方库，使用 glide 工具来管理依赖。 *可以忽略*
* web - 网页应用的特定组件，静态资源、服务端网页模板、单页应用等。 *可以忽略*
* website - 如果你是开源项目且不想使用 Github pages 的话，可以把项目的网站放在这里。 *可以忽略*

其中比较重要的如下：

### /pkg

这个目录中存放的就是项目中可以被外部应用使用的代码库，其他的项目可以直接通过 import 引入这里的代码，所以当我们将代码放入 pkg 时一定要慎重，不过如果我们开发的是 HTTP 或者 RPC 的接口服务或者公司的内部服务，将私有和公有的代码都放到 /pkg 中也没有太多的不妥，因为作为最顶层的项目来说很少会被其他应用直接依赖，当然严格遵循公有和私有代码划分是非常好的做法，作者也建议各位开发者对项目中公有和私有的代码进行妥善的划分。

### /internal

私有代码推荐放到 `/internal` 目录中，真正的项目代码应该写在 `/internal/app` 里，同时这些内部应用依赖的代码库应该在 `/internal/pkg` 子目录和 `/pkg` 中，下图展示了一个使用 `/internal` 目录的项目结构：
![](https://img.draveness.me/golang-internal-app-and-pkg.png)

### /cmd

`/cmd` 目录中存储的都是当前项目中的可执行文件，该目录下的每一个子目录都应该包含我们希望有的可执行文件，如果我们的项目是一个 grpc 服务的话，可能在 `/cmd/server/main.go` 中就包含了启动服务进程的代码，编译后生成的可执行文件就是 `server`。

我们不应该在 `/cmd` 目录中放置太多的代码，我们应该将公有代码放置到 `/pkg` 中并将私有代码放置到 `/internal` 中并在 `/cmd` 中引入这些包，保证 `main` 函数中的代码尽可能简单和少。

### /api
`/api` 目录中存放的就是当前项目对外提供的各种不同类型的 API 接口定义文件了，其中可能包含类似 `/api/protobuf-spec`、`/api/thrift-spec` 或者 `/api/http-spec` 的目录，这些目录中包含了当前项目对外提供的和依赖的所有 API 文件：

```
$ tree ./api
api
└── protobuf-spec
    └── oceanbookpb
        ├── oceanbook.pb.go
        └── oceanbook.proto
```

二级目录的主要作用就是在一个项目同时提供了多种不同的访问方式时，用这种办法避免可能存在的潜在冲突问题，也可以让项目结构的组织更加清晰。

### Makefile

最后要介绍的 Makefile 文件也非常值得被关注，在任何一个项目中都会存在一些需要运行的脚本，这些脚本文件应该被放到 /scripts 目录中并由 Makefile 触发，将这些经常需要运行的命令固化成脚本减少『祖传命令』的出现。

### 其它


* configs - 配置文件
* docs - 设计文档
* examples - 应用程序或者公共库使用的一些例子
* scripts - 其他一些脚本，编译、安装、测试、分析等等
* test - 功能测试，性能测试，测试需要的数据等
* tools - 常用的工具和脚本，可以引用 `/internal` 或者 `/pkg` 里面的库
* src - 绝对不应该存在的目录，如果 Go 项目有这个文件夹，多数都是 Java 转来的开发者。

### cmd, internal 和 pkg

Go 代码只存在于这三个文件夹。

* cmd - 不要把很多的代码放到这，这里的代码尽可能简单。
  * 通常来说 cmd 目录下的文件都非常小，它的主要作用就是引入和调用 pkg 和 internal 的代码。
  * 可以有子文件夹，各自拥有 main.go
  * 编译的话，可以在根目录执行 `go run cmd/binaryname/main.go`
* pkg - 可供外部项目使用的代码，代码放在这里前请三思。
* internal - 如果代码不可重用，或者不想被其它项目使用。换句话说就是项目的专属代码。
  * 某个应用私有的代码放到 `/internal/myapp/` 目录下
  * 多个应用通用的公共的代码，放到 `/internal/common` 之类的目录

### 例子

```bash
├── cmd/
│   └── binaryname/
│       └── main.go # a small file glueing things together
├── internal/
│   └── data/
│       ├── types.go
│       └── types_test.go # unit tests are right here
├── pkg/
│   └── api/
│       └── types.go  # REST API input and output types
├── test/
     └──  smoketest.py
```

```bash
├── cmd/
│   └── app/
│       ├── main.go
│       └── main_test.go
├── go.mod
├── go.sum
├── pkg/
    ├── handlers
    │   └── handlers.go
    ├── types
    │   └── types.go
    └── util
        └── util.go
```

## 相关讨论

* [Best project structure when using go 1.11 mod](https://github.com/golang-standards/project-layout/issues/18)
