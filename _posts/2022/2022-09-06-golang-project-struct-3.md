---
layout: post
title:  "Golang 项目尝试使用六边形架构"
date:   2022-09-06 06:10:00

categories: go
tags: refactoring
author: "Victor"
---

## 传统架构的问题

1. 应用逻辑在不同层泄露，导致替换某一层变得困难、难以对核心逻辑完整测试。常常会出现业务逻辑写在了表示层，或者业务逻辑直接和数据访问绑定。
2. 传统的分层架构是一维的结构，有时应用不光是上下的依赖，可能是多维的依赖，这时一维的结构就无法适应了。

## 六边形架构的理解

分层系统是一种架构风格，本质是避免耦合的出现。其核心理念就是应用通过端口与外部进行交互的。核心的业务逻辑（领域模型）与外部资源（数据库等资源）完全隔离，仅通过适配器进行交互，解决了业务逻辑与用户数据交错的问题，很好的实现了前后端分离。

所有数据处理全部由 repository 适配成实体，逻辑都是领域服务、聚合、实体的行为。分多少层和平层、跨层调用本身也不存在。

### 六边形的优点

1. 自动测试。自动化测试成为设计第一要素，同时自动化测试也保证应用逻辑不会泄露到用户界面，在技术上保证了层次的分界。
2. 外部可替换。一个端口对应多个适配器，应用通过端口为外界提供服务，这些端口需要被良好的设计和测试。内部不关心外部如何使用端口，从一开始就要假定外部使用者是可替换的。一般端口数不会超过4个。
3. 关注点分离。重心放在业务逻辑上，外部的驱动逻辑或被驱动逻辑存在可变性、可替换性，依赖具体技术细节。而业务逻辑相对更加稳定，体现应用的核心价值，需要被详尽的测试。
4. 依赖倒置。不能出现内部依赖外部的情况，只能外部依赖内部，这样才能保证外部是可以替换的。被驱动者适配器，实际是内部依赖外部，这时需要使用依赖倒置，由驱动者适配器将被驱动者适配器注入到应用内部，这时端口的定义在应用内部，但是实现是由适配器实现。

### 目录设计方案 1

```bash
1. domain - 领域模型
  * aggregate - 聚合
  * entity - 实体
  * dto - 传输对象
  * po - 持久化对象
  * *.go - 领域服务
2. adapter - 端口适配器
  * controller - 输入适配器
  * repository - 输出适配器
3. server - 服务端程序入口
  * conf - 配置文件
  * main.go - 主函数
4. infra - 基础设施
  * *.go - 基础设施组件
```

### 目录设计方案 2

```bash
1. adaptor - 入口适配器，实现 facade 层的接口。
  * http
  * rpc
  * consumer
2. infrastructure - 出口适配器，依赖 domain，application 层，根据业务场景定制接口
  * cache 缓存
  * producer 消息生产
  * repository 数据库
  * rpc
  * task 任务调度
3. application - 应用层，依赖 domain，facade，体现对外功能实现。判断当前功能性质，校验并判断，执行不同的操作，承上启下。
4. bootstrap 启动类、配置文件、集成测试
5. common 工具类,枚举类,异常,跨模块通用对象
6. domain 领域层, 处理主要的业务逻辑问题,如果逻辑复杂,可扩展子模块
7. facade 外观层, 对外提供所有的接口定义
```

## 努力的方向

MVC 和传统 4 层架构中，infrastructure 基础设施层封装了 redis 和 mq 的调用逻辑，这造成 domain 依赖 infrastructure。有一些功能会写在 service 里面，比如：

* 执行成功就发个 mq
* 执行某个事件的时候，同步一下缓存。

domain 是写业务逻辑的，优化不涉及业务逻辑的改动，但优化需要改动 domain 层，可见上面这种分层架构方案是错的。应该使用依赖倒置的方案进行优化。具体的做法如下：

1. domain 层定义储存的接口，比如 XXRepository，不写具体实现。
2. domain 层不依赖其它包的类，用到数据存储时，调用 domain 层抽象的接口即可。
3. 高层通过依赖注入的方式，将基础设施的实现传入到 domain 层中。

最终实现，抽象全部在 domain 层，细节全往 application 和 infrastructure 去堆。如果再出现修改对象，需要同步更新缓存的场景，就可以让 application 去完成这个组装工作。

## 参考

* [六边形架构理解 - 知乎](https://zhuanlan.zhihu.com/p/378085465)
* [【六边形架构示例】 - 简书](https://www.jianshu.com/p/f42881f01971)
* [六边形架构和分层架构的区别-51CTO.COM](https://www.51cto.com/article/607776.html)

### 推荐

* [Hexagonal Architecture in Go - DEV Community 👩‍💻👨‍💻](https://dev.to/prinick/hexagonal-architecture-in-go-3i7n)
