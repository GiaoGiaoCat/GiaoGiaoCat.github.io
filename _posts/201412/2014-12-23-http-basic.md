---
layout: post
title:  "HTTP 下午茶节选"
date:   2014-12-23 11:20:00
categories: Other
tags: learningnote
author: "Victor"
---

## 内容节选

### URL 编码(URL Encoding)

URL 在设计的时候就默认只接受 ASCII 码。因此，不安全的或者不是 ASCII 码的字符就要进行转义或者编码来适应这个格式。

URL 编码的原理是将不符合格式的字符替换成 ```%``` 开头后面跟着两个十六进制数字代表的 ASCII 码的一串字符。

下面是一些常见的 URL 编码和实例 URL：

字符 | ASCII 码 | URL
---|---|---
space|020|http://www.thedesignshop.com/shops/tommy%020hilfiger.html
!|041|http://www.thedesignshop.com/moredesigns%041.html
+|053|http://www.thedesignshop.com/shops/spencer%053.html

#### 符合下列条件的字符都要进行编码处理：

1. 没有对应的 ASCII 码。
2. 字符的使用是不安全的。比如 ```%``` 就是不安全的，因为它经常用于对其它字符进行转义。
3. 字符是有特殊用途的 URL 模式保留字。有些字符用于保留字是有着特殊的意义；它们在 URL 中的存在具有特殊用途。比如 ```/，?，:，&```，都是需要进行编码的保留字。

#### 在 URL 里安全使用的字符：

只有字母表里的和 ```$-_.+!'()，"``` 这些字符可以。

### HTTP 工具

#### 浏览器插件

* [Postman](https://chrome.google.com/webstore/search/Postman?hl=en-US)
* [HTTP API Client](https://chrome.google.com/webstore/detail/dhc-resthttp-api-client/aejoelaoggembcahagimdiliamlcdmfm)

#### GUI 工具

* [Paw2](http://luckymarmot.com/paw)
* [HTTP Client](http://ditchnet.org/httpclient/)
* [Fiddler](http://www.telerik.com/fiddler)
* [Cocoa Rest Client](http://ditchnet.org/httpclient/)

#### 命令行工具

* [curl](http://curl.haxx.se/)

## 处理响应

HTTP 响应中最重要的部分如下:

* 状态码
* 头部
* 消息正文，里面有原始响应数据

302 Redirect（重定向）当一个资源的位置移动了，把对旧 URL 的请求重新引导到新 URL 上。

### 请求头部

字段名 | 描述 | 举例
---|---|---
Host|服务器域名|Host:www.reddit.com
Accept-Language|可接受的语言|Accept-Language: en-US，en;q=0.8
User-Agent|一个标识客户端的字符串|User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML，like Gecko) Chrome/38.0.2125.101 Safari/537.36
Connection|客户端连接的类型|Connection: keep-alive
Content-Encoding|数据的编码类型|Content-Encoding: gzip
Server|服务器的名称|Server:thin 1.5.0 codename Knife
Location|通知客户端新的资源位置|Location: http://www.github.com/login
Content-Type|响应数据的类型|Content-Type:text/html; charset=UTF-8

## 有状态的 web 应用

HTTP 协议是无状态的。换句话说，在你的各次请求之间，服务器是不会保留你的 **状态** 信息。

会话数据是由服务器生成并存储在服务器上，会话 id 以 cookie 的形式发送到客户端上。我们还看到了 web 应用程序如何充分利用这些来模拟在 web 上的有状态体验。

## 安全

* HTTPS 通过一个叫做 TLS 的加密协议来加密消息。在 TLS 开发完成前，早期 HTTPS 使用 SSL （ Secure Sockets Layer ）。
* 通过 HTTPS 发送的请求和响应在发送前都会被加密。
* 同源策略是一个重要的概念，它允许来自同一站点的资源进行互相访问而不受限制，但是会阻止其他不同站点对文档/资源的访问。
* CORS 是一种机制，允许我们绕过同源策略，从一个域名向另一个域名的资源发起请求。CORS 的原理是添加新的 HTTP 头部，来对一些域名授权，那这些域名就可以发起对本页面资源的请求。

## 原文

* http://happypeter.github.io/tealeaf-http/
* http://i5ting.github.io/tealeaf-http/preview
