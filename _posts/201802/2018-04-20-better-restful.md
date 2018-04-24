---
layout: post
title:  "RESTful & JSON API"
date:   2018-04-20 12:00:00

categories: tool
tags: api
author: "Victor"
---

正在读《RESTful Web APIs中文版》之后会再次补充此文。

## 前提

**永远不要跟业界通行的方法背着干，这不是个人偏好的问题，是项目生死的大事。**

所谓代码风格、接口形式、各种林林总总的格式规定，其实都是为了在团队内部形成共识、防止个人习惯差异引起的混乱。

JSON-RPC 当然也是有规范的，但相比 REST 实在宽松太多了。

注意：多数时候我们提到的规范一词都是 specification 的意思，不是 regulation 的意思，其本身并没有约束力。

## REST

这一段太长不想看，那只要看这一句 **就是用 URL 定位资源，用 HTTP METHOD 描述操作，看 HTTP STATUS CODE 就知道结果如何。**

REST -- Resource Representational State Transfer 直接翻译：资源在网络中以某种表现形式进行状态转移。抱歉，这个中文我完全不懂什么意思。

* Resource：资源，可以是一段文本、图片、歌曲或者 **一种服务**。比如 newsfeed，friends 等；
* Representational：某种表现形式，比如 JSON，XML，JPEG 等；
* State Transfer：状态变化。通过 HTTP 动词实现。
  * 取东西就要 GET - GET就是安全的，不会修改服务资源
  * 新增就要 POST - POST就是不安全的
  * 修改就要 PUT - PUT就要幂等
  * 删除就是 DELETE - DELETE就要幂等

REST 的使用场景是 Machine-to-machine 的系统集成，目标是让服务发布者和消费者在最小约束下自由演化。

这个约束是指服务契约，简单讲就是服务输入输出的语义。消费者只需知道服务的根资源的 URI，就可以由根资源引导到所需的资源。换句话说，消费者和发布者的耦合只在于根资源的 URI 以及各资源及其操作的语义。

## RESTful API

一个架构符合 REST 原则，就称它为 RESTful 风格。

很多人在尚未真正理解 REST 的一些核心概念的情况下，就到处吹自己设计的 API 是 RESTful API，大佬 Fielding 忍无可忍发了一篇博客意思是，RESTful API 必须是超文本驱动的，也就是 HATEOAS 这个概念。

**只要把 HTTP 标准和扩展的 RFC 看完，就会觉得 RESTful 就是按 HTTP 说的那样用。只是很多人的 HTTP 没按 HTTP 说的那样用。**

### RESTful API 设计指南

每个人都想从头开始设计他们自己的 API，即使从业务的角度看并没什么意义。

详细看 [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

另外注意一下：

* 每一个 URI 代表一种资源，独一无二
* 资源的地址推荐用嵌套结构
* 警惕返回结果的大小。HTTP 协议支持分页 Pagination 操作，在 Header 中使用 Link 即可
* 在返回结果用明确易懂的文本。注意返回的错误是要给人看的，避免用 1001 这种错误信息

### 有哪些好处

总的来说就是 **简单、低耦合**，因为在复杂、紧耦合的情况下事情更容易失控。

举例来说好处的话，大概有这么几条：

* 透明性，暴露资源存在，面向资源，服务自解释
* 充分利用 HTTP 协议本身语义。
* 无状态，幂等。这点非常重要。在调用一个接口（访问、操作资源）的时候，可以不用考虑上下文，不用考虑当前状态，极大的降低了复杂度。
* HTTP 本身提供了丰富的内容协商手段，无论是缓存，还是资源修改的乐观并发控制，都可以以业务无关的中间件来实现。
* 降低消费者对服务内部实现细节的耦合

### HTTP 状态码

2XX/3XX 都是请求成功，但是结果不同。4XX 是请求出错，5XX 是服务器处理出现错误。

![](https://pic4.zhimg.com/80/v2-35a9a08efff828645c5d980f0166c832_hd.jpg)

相关阅读：

* [关于 RESTful API 中 HTTP 状态码的定义的疑问](https://www.zhihu.com/question/58686782/answer/159603453)
* [10 Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
* [List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#cite_note-15)
* [Choosing an HTTP Status Code — Stop Making It Hard](http://racksburg.com/choosing-an-http-status-code/)

### RESTful 常见的一些问题

1. `login` 和 `logout` 应该怎么 REST 化
2. 多条件复合搜索在 GET 里写不下怎么办
3. 大量资源的删除难道要写几千个 `DELETE`

其实在理解了 REST 后，这些都不是什么无解的难题，只是思维方式要转换一下：

1. `login` 和 `logout` 只是对 `session` 资源的创建和删除
2. search 本身就是个资源，使用 POST 创建，如果不需持久化，可以直接在 Response 中返回结果，如果需要（如翻页、长期缓存等），直接保存搜索结果并 303 跳转到资源地址就行
3. id 多到连 url 都写不下的请求，应该创建 task，用 GET 返回 task 状态甚至执行进度

## JSON-RPC vs RESTful API

### RPC

RPC 的思想是把本地函数映射到 API，也就是说一个 API 对应的是一个 function，我本地有一个 `getAllUsers`，远程也能通过某种约定的协议来调用这个 `getAllUsers`至于这个协议是 Socket、是 HTTP 还是别的什么并不重要；

RPC中的主体都是动作，是个动词，表示我要做什么。

### RESTful

URL 主体是资源，是个名词。而且也仅支持 HTTP 协议，规定了使用 HTTP Method 表达本次要做的动作，类型一般也不超过那四五种。这些动作表达了对资源仅有的几种转化方式。

### 比较

两者没有高下之分，无非是一种约定俗成的标准。习惯用RPC就用RPC，能理解REST就用REST。

* JSON-RPC 比较符合直观，格式也相对宽松
* REST 流行，有设计规范

JSON-RPC 无法像 REST 一样享受 HTTP 的各种优点(standard interface, stateless, cache..)，又必须承担 HTTP 作为基于文本的协议，payload 过大传输的成本以及序列化反序列化的开销。

如果你想寻求一种 RPC 框架，Thrift 或 protobuf 无疑更合适。

## RFC

Request For Comments（RFC），是一系列以编号排定的文件。文件收集了有关因特网相关资讯，以及 UNIX 和因特网社群的软件文件。

所有关于 Internet 的正式标准都是以 RFC 文档形式出版。但大量的 RFC 文档都不是正式的标准，出版目的都是为了提供信息。

RFC 文档在成为官方标准前一般至少要经历 4 个阶段：因特网草案、建议标准、草案标准、因特网标准。

[RFC文档目录](http://man.chinaunix.net/develop/rfc/default.htm)

## 相关阅读

* [Architectural Styles and the Design of Network-based Software Architectures](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [从消费者的角度评估REST的价值](http://hippoom.github.io/blogs/value-of-hypermedia-from-client-perspective.html)
* [理解本真的REST架构风格](http://www.infoq.com/cn/articles/understanding-restful-style)
* [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful)
* [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
* [RESTful风格的优势是什么](https://blog.csdn.net/wlchn/article/details/48369233)
* [WEB开发中，使用JSON-RPC好，还是RESTful API好](https://www.zhihu.com/question/28570307/answer/163638731)
* [举例说明，RESTful 到底有哪些好处？](https://www.zhihu.com/question/20130130)
