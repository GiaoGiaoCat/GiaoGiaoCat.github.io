---
layout: post
title:  "JWT"
date:   2018-04-10 15:05:00

categories: other
tags: api
author: "Victor"
---

## 前言

随着移动互联网时代到来，客户端的类型越来越多， 逐渐出现了一个服务器，N个客户端的格局。

### 本文不讨论

* 如何提供第三方调用
* HTTP 协议明文传输信息而造成的安全问题
* 如何保证请求的唯一性，避免重放攻击

### session 和 cookie

传统的用户登录认证中，因为 http 是无状态的，所以都是采用 session 方式。用户登录成功，服务端会保存一个 session，给客户端一个 sessionId，客户端会把 sessionId 保存在 cookie 中，每次请求都会携带这个 sessionId。

* [How Rails Sessions Work](https://www.justinweiss.com/articles/how-rails-sessions-work/)
* [详解SESSION与COOKIE的区别](http://blog.sina.com.cn/s/blog_59e16a4d0100q3yn.html)
* [Cookie/Session机制详解](https://blog.csdn.net/fangaoxin/article/details/6952954)
* [理解Cookie和Session机制](https://my.oschina.net/xianggao/blog/395675)

## JWT Token

Token 说到底也就是个字符串，重点是这个字符串该怎么写才会比较合理。

### 优势

* 简洁 Compact：数据量小
* 自包含 Self-contained：在后续请求中减少查询数据库的几率
* 能保证令牌无法伪造

### 介绍

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2018-04-10-api-jwt-token/20180227200506327.png)

JWT（Json Web Token）本质上是一种 Token 的设计规范，RFC 7519。

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

### 单点登录和账号互踢

先说简单的情况，同一账户所有设备共用一个 token：

1. 登录时候在 Redis 中存储 string 键值对。键 `<namespace>:tokens:<user-id>` 值 `<token>`
2. 再次登录和退出时，更新 值 `<token>` 或删除 键 `<namespace>:tokens:<user-id>`
3. 可以选择使用第三方即时通讯 SDK 发送广播通知其他客户端下线
4. 其他客户端收到通知清除本地 token 显示相关信息并跳转到登录页面

复杂点的情况，同一账户同类型设备共用一个 token：

1. 登录时候在 Redis 中存储 string 键值对。键 `<namespace>:tokens:<device-type>:<user-id>` 值 `<token>`
2. 其他步骤同上

希望能显示用户被什么设备踢下线。可以在 User 表中添加 `sign_in_count, failed_sign_in_count, current_sign_in_at, current_sign_in_ip, last_sign_in_at, last_sign_in_ip, last_failed_sign_in_at, last_changed_by, last_changed_at` 等字段。

也可以在 Redis 中设计一个 Hash，键名称可以为 `<namespace>:users-logging:<user-id>` 储存这些值。

并在每次登录的时候进行记录这些。

优势：

* 也可以实现用户管理自己的登录设备，删除指定的 token 就像手机微信登出网页版一样
* token 写入和查询不操作数据库

### 数据量大小是否方便传输

假如我们的 payload 如下

```
{
 "sub": "1234567890",
 "name": "John Doe",
 "admin": true,
 "jti": "8bf764e7-c887-4c7d-9e35-aa95ea65a57a",
 "iat": 1523454554,
 "exp": 1523458154
}
```

生成的 token 体积是 0.6 KB。这点数据量，算啥啊。

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

* [rfc7519](https://tools.ietf.org/html/rfc7519)
* [Life in a post-database world: using crypto to avoid DB writes](https://neosmart.net/blog/2015/using-hmac-signatures-to-avoid-database-writes/) 这篇文章很棒，值得一读
* [Introduction to JSON Web Tokens](https://jwt.io/introduction/)
* [CAS](https://www.apereo.org/projects/cas)
* [10 Things You Should Know about Tokens](https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/)
* [ActionController::HttpAuthentication::Token](http://api.rubyonrails.org/classes/ActionController/HttpAuthentication/Token.html)
* [Building a simple token based authorization API with Rails](https://engineering.musefind.com/building-a-simple-token-based-authorization-api-with-rails-a5c181b83e02)
* [Token Based Authentication in Rails](https://www.codeschool.com/blog/2014/02/03/token-based-authentication-rails/)
* [理解JWT的使用场景和优劣](http://blog.didispace.com/learn-how-to-use-jwt-xjf)

### Gems

* [knock](https://github.com/nsarno/knock) Seamless JWT authentication for Rails API
* [ruby-jwt](https://github.com/jwt/ruby-jwt) A pure ruby implementation of the RFC 7519 OAuth JSON Web Token (JWT) standard.
* [Rack middleware for blocking & throttling](https://github.com/kickstarter/rack-attack)
