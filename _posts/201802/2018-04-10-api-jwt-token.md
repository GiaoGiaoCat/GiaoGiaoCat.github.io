---
layout: post
title:  "API User Token 设计"
date:   2018-04-10 15:05:00

categories: rails
tags: advanced
author: "Victor"
---

## 前言

随着移动互联网时代到来，客户端的类型越来越多， 逐渐出现了一个服务器，N个客户端的格局。

### 本文不讨论

* 如何提供第三方调用
* HTTP 协议明文传输信息而造成的安全问题
* session 引起的 CSRF 跨站请求伪造的攻击的问题

### session 和 cookie

传统的用户登录认证中，因为 http 是无状态的，所以都是采用 session 方式。用户登录成功，服务端会保存一个 session，给客户端一个 sessionId，客户端会把 sessionId 保存在 cookie 中，每次请求都会携带这个 sessionId。

## JWT

### 优势

* 简洁 Compact：数据量小
* 自包含 Self-contained：减少了需要查询数据库的需要
* 能保证令牌无法伪造

### 介绍

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2018-04-10-api-jwt-token/20180227200506327.png)

Token 说到底也就是个字符串，重点是这个字符串该怎么写才会比较合理。JWT（Json Web Token）本质上是一种 Token 的设计规范，RFC 7519。

整体结构为：

* header(base64) 使用的签名算法
* payload(base64) 负载，存放有效必要的非敏感信息
  * 标准中注册的声明
  * 公共的声明
  * 私有的声明
* Signature 使用编码后的 header 和 payload 以及一个秘钥进行签名

```bash
# Header
{
  "alg": "HS256",
  "typ": "JWT"
}

# Payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

# Signature
{
  HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
}
```

JWT 的过期和刷新也很好做，但设计上无法做到服务端禁用，参考业界主流做法，AWS、Azure 和 Auth0 都是用 JWT 为载体，ID Token + Access Token + Refresh Token 的模式：

* [Using Tokens with User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)
* [Azure AD token reference](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-token-and-claims)
* [Tokens used by Auth0](https://auth0.com/docs/tokens)

### 关于 JWT 的 Claim

也就是 payload 中可携带的 **标准中注册的声明**，建议但不强制使用。

其中 `exp, nbf, iat, jti` 时间戳确保 Token 不是一直有效。

| Reserverd claims | 说明
|---|---|
| iss(Issuser) | 代表这个 JWT 的签发主体 |
| sub(Subject) | 代表这个 JWT 的主体，即它的所有人 |
| aud(Audience) | 代表这个 JWT 的接收对象 |
| exp(Expiration time) | 是一个时间戳，代表这个 JWT 的过期时间 |
| nbf(Not Before) | 是一个时间戳，代表这个 JWT 生效的开始时间，意味着在这个时间之前验证 JWT 是会失败的 |
| iat(Issued at) | 是一个时间戳，代表这个 JWT 的签发时间 |
| jti(JWT ID) | 是 JWT 的唯一标识 |

### 服务端禁用

用户登录我们需要创建 `token -> uid` 的关系，用户退出时需要需删除 `token -> uid` 的关系。用户更新密码时我们也要更新对应的关系。
查询 redis 缓存中的 uid，如果获取不到这说明该 token 已过期或被服务端禁用。

## 关于 Signature

### 单向散列函数

单向散列函数也称信息摘要函数 Message Digest Function, 哈希 Hash 函数或者杂凑函数。很常见的一个应用就是文件的校验。

你可以简单的理解为：

1. 输入：任意长度消息（可以是1bit 、1K、1M，甚至可以100G）
2. 输出：一串固定长度的数据（散列值，也称消息摘要，指纹）

单项散列函数有如下几种特性：

* 根据任意长度消息得出的散列值长度是固定的
* 散列值计算时间短
* 不同的消息有不同的散列值（如果两个不同消息的散列值相同，那就称为发生碰撞）
* 根据散列值无法还原消息（单向性，只能从消息到散列值，反之不成立）

### MAC 消息认证码

Message Authentication Code 即消息认证码 它可以确认消息完整性并进行认证。

1. 输入：消息+发送者以及接收者之间共享的密钥（与单项散列函数不同之处就是它多了一个密钥的参与）
2. 输出：固定长度的数据，称为MAC值（跟单向散列函数一样）

确切来说它指的是一种认证机制。这种机制有多种实现方法，单项散列函数就是其中之一。使用单向散列函数（也称Hash函数）实现的消息认证码就称为 HMAC，其中H就是Hash的意思。

它解决了单项散列函数虽然可以检测到篡改（完整性），但是却没有办法识别伪装的问题。

### HMAC 哈希消息认证码

HMAC 就是使用了单项散列函数来构造消息认证码的一种方法（RFC2104）。根据它所使用的散列函数不同，就出现了如下这么多种 HMAC

| HMAC 算法 | 备注
|---|---|
| HS256 | HMAC using SHA-256 |
| HS384 | HMAC using SHA-384 |
| HS512 | HMAC using SHA-512 |

**使用消息认证码是无法保证消息的机密性的，它只能保证消息被正确的传送了。**

例如 传送的完整消息格式为 `123456 + 消息验证码`，消息验证码的作用就是在对方收到消息之后可以根据验证码来验证 `123456` 是没有被修过的。至于机密性，那需要对 `123456` 进行加密，而这不关消息验证码什么事。

### 小结

JWT 中被称为 Signature 的部分，其实是前两部分的消息认证码 MAC。在 JWT 中密钥并不与客户端共享，其为服务器独有。这样一来只有服务器可以发 Token，而客户端因为缺少密钥而无法伪造 Token。


## 相关链接

* [Introduction to JSON Web Tokens](https://jwt.io/introduction/)
* [CAS](https://www.apereo.org/projects/cas)
* [10 Things You Should Know about Tokens](https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/)
* [ActionController::HttpAuthentication::Token](http://api.rubyonrails.org/classes/ActionController/HttpAuthentication/Token.html)
* [Building a simple token based authorization API with Rails](https://engineering.musefind.com/building-a-simple-token-based-authorization-api-with-rails-a5c181b83e02)
* [Token Based Authentication in Rails](https://www.codeschool.com/blog/2014/02/03/token-based-authentication-rails/)

### Gems

* [knock](https://github.com/nsarno/knock) Seamless JWT authentication for Rails API
* [ruby-jwt](https://github.com/jwt/ruby-jwt) A pure ruby implementation of the RFC 7519 OAuth JSON Web Token (JWT) standard.
* [Rack middleware for blocking & throttling](https://github.com/kickstarter/rack-attack)
