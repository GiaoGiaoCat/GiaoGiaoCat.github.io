---
layout: post
title:  "Go 和 WebSocket"
date:   2019-04-27 13:00:00

categories: go
tags: advanced
author: "Victor"
---

## 什么是 WebSocket

WebSocket 是一种在单个 TCP 连接上进行全双工通信的协议。WebSocket 通信协议于 2011 年被IETF定为标准 RFC 6455，并由 RFC7936 补充规范。WebSocket API 也被 W3C 定为标准。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

### 背景

传统网站为了实现推送功能，所用的技术都是轮询。轮询是在特定的的时间间隔，由浏览器对服务器发出 HTTP 请求，然后由服务器返回最新的数据给客户端的浏览器。这种模式有明显的缺点，即浏览器需要不断的向服务器发出请求，然而 HTTP 请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

而比较新的技术去做轮询的效果是 Comet。这种技术虽然可以双向通信，但依然需要反复发出请求。而且在 Comet 中，普遍采用的长链接，也会消耗服务器资源。

### WebSocket 优点

1. 较少的控制开销。
  * 在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小。在不包含扩展的情况下，对于服务器到客户端的内容，此头部大小只有2至10字节（和数据包长度有关）
  * 对于客户端到服务器的内容，此头部还需要加上额外的4字节的掩码。相对于 HTTP 请求每次都要携带完整的头部，此项开销显著减少了。
2.  更强的实时性。
  * 由于协议是全双工的，所以服务器可以随时主动给客户端下发数据。相对于 HTTP 请求需要等待客户端发起请求服务端才能响应，延迟明显更少
  * 即使是和 Comet 等类似的长轮询比较，其也能在短时间内更多次地传递数据。
3.  保持连接状态。
  * 与 HTTP 不同的是，Websocket 需要先创建连接，这就使得其成为一种有状态的协议，之后通信时可以省略部分状态信息。而 HTTP 请求可能需要在每个请求都携带状态信息（如身份认证等）。
4.  更好的二进制支持。
  * Websocket 定义了二进制帧，相对 HTTP，可以更轻松地处理二进制内容。
5. 可以支持扩展。
  * Websocket 定义了扩展，用户可以扩展协议、实现部分自定义的子协议。如部分浏览器支持压缩等。
6. 更好的压缩效果。
  * 相对于 HTTP 压缩，Websocket 在适当的扩展支持下，可以沿用之前内容的上下文，在传递类似的数据时，可以显著地提高压缩率。

### 握手协议

* WebSocket 是独立的、创建在 TCP 上的协议。
* Websocket 通过 HTTP/1.1 协议的101状态码进行握手。
* 为了创建 Websocket 连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（handshaking）。

### 如何工作

* Web 浏览器和服务器都必须实现 WebSockets 协议来建立和维护连接。由于 WebSockets 连接长期存在，与典型的 HTTP 连接不同，对服务器有重要的影响。
* 基于多线程或多进程的服务器无法适用于 WebSockets，因为它旨在打开连接，尽可能快地处理请求，然后关闭连接。任何实际的 WebSockets 服务器端实现都需要一个异步服务器。

WebSocket 的协议在第一次 handshake 通过以后，连接便建立成功，其后的通讯数据都是以 `\x00` 开头，以 `\xFF` 结尾。在客户端，这个是透明的，WebSocket 组件会自动将原始数据掐头去尾。

## Go 语言实现 Websocket

由于 Go 语言标准包里面没有对 WebSocket 的支持，但是官方维护的 go.net 对这个有支持，所以可以获取 `go get golang.org/net/websocket`。

## 相关文章

* [WebSocket Echo Test](https://www.websocket.org/echo.html) 一个用来测试 WebSocket 的站点
* [WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)
* [WebSocket 教程](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
* [WebSocket 详解教程](https://www.cnblogs.com/jingmoxukong/p/7755643.html)
