---
layout: post
title:  "Golang 项目架构中的一点思考"
date:   2019-12-02 17:10:00

categories: go
tags: refactoring
author: "Victor"
---

在阅读了 [project-layout](/go/go-project-structure/) 和 [EBI 架构模式](https://ebi.readthedocs.io/en/latest/) 之后，目前的项目大概是这个样子的。

```bash
.
├── .air.conf                # Config file for air.
├── Dockerfile
├── Makefile
├── README.md
├── apiary.apib              # API Documentation.
├── cmd                      # Main applications for this project.
│   ├── migration            # Database migration.
│   └── server
│       ├── main.go          # Entry point of the API Server.
│       └── routes           # Registering route handlers and middleware.
├── configs                  # Contains all configuration files.
├── docs                     # Design and user documents.
├── go.mod
├── go.sum
├── internal                 # Private application and library code.
│   ├── consts               # Provides global constants.
│   ├── entity               # Core Layer & Data Serialization and Deserialization.
│   ├── handler              # Presentation Layer, defines handler methods of endpoints.
│   ├── repository           # Persistence Layer.
│   ├── service              # Interactors Layer.
│   └── structure            # Data Transfer Object.
│       ├── request
│       └── response
├── pkg                      # Library code that's ok to use by external applications.
│   └── initializer          # Application configuration loader.
├── test                     # Additional external test apps and test data.
└── web                      # Web application specific components: static web assets, server side templates and SPAs.
```

### 原因

采用 **根据模块归组** 方法出现的问题：假设有 `users` 包，会出现 `users.User` 这样很傻的命名，而且很容易产生循环依赖的问题。

## 遇到的问题

1. 部分 Service 层会调用其它的 Dao 层, 比如 UserService 会调用 UserPositionDao 和 UserBookDao
2. 部分 Service 层会调用其它的 Service 层, 比如 UserService 会调用 UserAppleService 和 UserPencilService
3. 部分 Controller 层会调用多个 Service 层

### 尝试解决

J2EE 分层设计是 Java 企业应用的最基本的设计思想。从最常规的分层结构来说，系统层次从上到下依次为：

1. 表现层：主要是客户端的展示。
2. 服务层：直接为客户端提供的服务或功能。也是系统所能对外提供的功能。
3. 领域层：系统内的领域活动。
4. DAO层：数据访问对象，通过领域实体对象来操作数据库。

其中有些指导原则：

1. 上层总是依赖其下层，依赖关系不跨层。
2. 除表现层外，同一层之间方法不允许相互调用。如果同层发生调用，一定是调用的对外不可见的工具方法。
3. 从服务层出发进行设计，根据系统需要提供的功能进行分析，确定 Service 接口中的方法。**而不是从数据库的表出发，创建DAO，再创Domain，然后Service，这实际上是对系统分层的误解。**

在写代码的过程中多考虑这部分代码放在哪里，多里利用上下分层，增加代码可读性，提高代码复用率。

### Controller/Handler 层

* 调用 Service 逻辑设计层的接口来控制业务流程，接收前端参数，再将处理结果返回到前端。
* Controller 调用 Service 层是：一对一接口调用，且 Controller 层不做任何业务处理，目的是为了后续拓展直接替换 Controller 为 RPC 框架而准备。
* 一个 Controller 只能调用一个 Facade。

### Facade 层

* 处理 Service 的之间的复杂调用，不同 domain 可能以组合模式对外提供服务，就在这层。
* 一个 Facade 可以调用不同的 Service。

### Service 层

* 业务逻辑。
* 只能在 Service 中注入 DAO。
* 一个 Service 只调用一个 Dao, 控制 Service 调用 Dao 层的复杂度。
* Service 不能互相调用。
* 一般情况下事务放在 service 层，为了避免事务嵌套或单个事务过大等问题的。

### DAO/Repository 层

* 封装 Entity 对象，对数据库进行数据持久化操作，他的方法语句是直接针对数据库操作的，主要实现一些增删改查操作。
* DAO 只能操作单表数据。

### Entity/Model 层

* 实体层，用于存放我们的实体类，与数据库中的属性值基本保持一致。

## 相关

### 阅读

* [Golang 标准包布局](https://www.jianshu.com/p/022ba2dd9239)
* [facade 层，service 层，domain 层，dao 层设计](https://www.iteye.com/blog/fei-6666-446247)
* [Golang 设计模式相关源码](https://blog.csdn.net/jeanphorn/article/details/78077805)
* [golang 设计模式](https://github.com/BPing/golang_design_pattern)
* [超赞的GO语言设计模式和成例集锦](https://studygolang.com/articles/8230)

### EBI 相关

* [Clean Architecture using Golang](https://dev.to/codenation/clean-architecture-using-golang-5791)
* [Trying Clean Architecture on Golang](https://hackernoon.com/trying-clean-architecture-on-golang-2-44d615bf8fdf)
* [EBI 架构(译)](https://www.jianshu.com/p/395814410cf5)
* [什么是实体边界交互器架构](https://www.jdon.com/51390)
* [在 Golang 中尝试简洁架构](https://studygolang.com/articles/12909)
* [例 1](https://github.com/bxcodec/go-clean-arch)
* [例 2](https://github.com/eminetto/clean-architecture-go)
* [VIPER模式介绍](https://blog.csdn.net/jingcheng345413/article/details/54969302)
