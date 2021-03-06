---
layout: post
title:  "重放攻击 Replay Attacks"
date:   2018-06-10 12:00:00

categories: other
tags: securing api
author: "Victor"
---

SSL/TLS 加密原理决定，两次完全相同的请求，第二次肯定是无效的。

## 重放攻击
### 什么是重放攻击

**重复的会话请求就是重放攻击。**

重放攻击（Replay Attacks）又称重播攻击、回放攻击或新鲜性攻击（Freshness Attacks），是指攻击者发送一个目的主机已接收过的包，来达到欺骗系统的目的，主要用于身份认证过程，破坏认证的正确性。

### 重放攻击的危害

请求被攻击者获取，并重新发送给认证服务器，从而达到认证通过的目的。
我们可以通过加密，签名的方式防止信息泄露，会话被劫持修改。但这种方式防止不了重放攻击。

### 重放攻击的防御
#### 1. 时间戳验证
请求时加上客户端当前时间戳，同时签名（签名是为了防止会话被劫持，时间戳被修改），服务端对请求时间戳进行判断，如超过5分钟，认定为重放攻击，请求无效。
**时间戳无法完全防止重放攻击。**

#### 2. 序号
在客户端和服务端通讯时，先定义一个初始序号，每次递增。这样，服务端就可以知道是否是重复发送的请求。

#### 3. 提问与应答的方式
**我们一般采用这种方式来防御重放攻击。**

客户端请求服务器时，服务器会首先生成一个随机数，然后返回给客户端，客户端带上这个随机数，访问服务器，服务器比对客户端的这个参数，若相同，说明正确，不是重放攻击。
这种方式下，客户端每次请求时，服务端都会先生成一个挑战码，客户端带上应答码访问，服务端进行比对，若挑战码和应答码不对应，视为重放攻击。

#### 4. HTTPS 防重放攻击
对于 https，每个 socket 连接都会验证证书，交换密钥。攻击者截获请求，重新发送，因为 socket 不同，密钥也不同，后台解密后是一堆乱码，所以 https 本身就是防止重放攻击的，除非能复制 socket，或者进行中间人攻击。

## 参考链接

* [重放攻击 Replay Attacks](https://www.cnblogs.com/shijingjing07/p/6123664.html)
* [分享下重放攻击的概念](https://cnodejs.org/topic/557c354d16839d2d539362b6)
* [傳輸層安全性協定](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)
* [HTTPS 入门及如何防重放攻击](https://blog.csdn.net/wzhqazcscs/article/details/78270781)
* [走 https 了，还需要对参数加密以及防止重放攻击吗](https://www.oschina.net/question/1047640_2180952)
