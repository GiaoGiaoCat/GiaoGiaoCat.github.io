---
layout: post
title:  "UUIDs 和 ULIDs 的区别"
date:   2019-02-21 12:00:00

categories: other
tags: knowledge
author: "Victor"
---

## 普通的 ids 有啥问题

大多数 Web 应用使用数据库的自增 ID 作为对象的唯一标识。

```ruby
p1 = Person.create!
p1.id
# => 1

p2 = Person.create!
p2.id
# => 2
```

数据库可以生成顺序标识，因为它存储了一个在记录创建时递增的计数器。这种模式在数据库之外也很常见，有时我们需要手动分配标识，比如可能会在 Redis 实例中存储一个自定义计数器。

这个实现简单又有效，但随着数据量的增长它面临了一些问题：

* 不可能同时创建记录，因为每个插入都必须排队等待接收其 id
* 请求顺序 id 可能依赖网络请求的返回值，导致性能下降
* 提供顺序标识的数据很难横向扩展,你必须处理好不同服务器上的计数器不同步问题
* 具有计数器的节点很容易成为单点故障
* 顺序的 id 还会暴露你的数据量，也很容易被猜测到其它资源标识符

## UUIDs 是 web-scale 的

UUIDs 一看就跟传统的顺序 ids 有很大的不同。它是 128-bit 数字，通常表示为 32 个 16进制数（hexadecimal digits）。

```
123e4567-e89b-12d3-a456-426655440000
```

UUIDs 使用 [RFC4122](https://www.ietf.org/rfc/rfc4122.txt) 中定义的算法来实现。试图解决顺序 ids 中的如下问题：

* 不需要在节点之间共享状态或协调处理，就可以在任意数量的节点上生成 UUIDs
* 它们比顺序 ids 更不容易被猜测
* 它们不会泄漏你数据集的大小

问题在于，即便只有很小很小的机会，仍然可能在两个节点上生成一样的 UUID，也就是冲突。

### 多种类型的 UUID

RFC 4122中定义了五种UUID算法。它们分为两类:

1. 基于时间和随机性的算法。每次运行都会产生一个新的 UUID：
  * Type 4: 随机生成的标识。新项目推荐用这个方案。
  * Type 1: ID 包含主机的 MAC 地址和当前时间戳。不推荐使用，因为它们太容易猜测。
  * Type 2: 不常见，好像是为了过时的 RPC 而设计的
2. 基于名字的算法有点不同。对于给定的一组输入，它们总是产生相同的 UUID
  * Type 5: 使用 SHA-1 哈希生成 UUID。推荐使用。
  * Type 3: 使用 MD5 散列，不推荐使用，因为 MD5 太不安全

在 Ruby 中使用 `uuidtools` 这个 gem 生成除 type 2 之外的所有类型 UUID。

```ruby
# Code stolen from the uuidtools readme. :)
require "uuidtools"

# Type 1
UUIDTools::UUID.timestamp_create
# => #<UUID:0x2adfdc UUID:64a5189c-25b3-11da-a97b-00c04fd430c8>

# Type 4
UUIDTools::UUID.random_create
# => #<UUID:0x19013a UUID:984265dc-4200-4f02-ae70-fe4f48964159>

# Type 3
UUIDTools::UUID.md5_create(UUIDTools::UUID_DNS_NAMESPACE, "www.widgets.com")
# => #<UUID:0x287576 UUID:3d813cbb-47fb-32ba-91df-831e1593ac29>

# Type 5
UUIDTools::UUID.sha1_create(UUIDTools::UUID_DNS_NAMESPACE, "www.widgets.com")
# => #<UUID:0x2a0116 UUID:21f7f8de-8051-5b89-8680-0195ef798b6a>
```

## 更先进的 ULIDs

ULID 是唯一标识符的一个新选择。它看起来是下面的样子：

```
01ARZ3NDEKTSV4RRFFQ69G5FAV
```

实际上 ULID 使用两组 base32-encoded 数字组成：UNIX 时间戳和一个随机数。

```
01AN4Z07BY      79KA1307SR9X4MV3

|----------|    |----------------|
 Timestamp          Randomness
   48bits             80bits
```

这个结构简单又有效，因为 UUID 的算法决定了它只能在时间戳或随机数中选择一种。成年人不做选择，ULIDs 都要！

因此 ULIDs 有一些有趣的特性：

* 它们可以按照字典（也就是字母序）排序
* 时间戳精确到毫秒
* 比 UUID 更漂亮

我们可以基于它搞出一些很酷的玩法：

* 如果要按日期对数据库进行分区，可以使用 ULID 中嵌入的时间戳来选择正确的分区
* 如果毫秒精度是可以接受的，可以按 ULID 排序，而不是单独的 `created_at` 列

它也有一些缺点：

* 如果公开时间戳对你来说不可接受，ULID 就不行了
* 如果你的排序精度要比 毫秒 还高，至少就不能按照 ULID 字段来排序了
* 有可能一些开源的 ULID 类库的实现不完整

## 原文

* [Going deep on UUIDs and ULIDs](https://blog.honeybadger.io/uuids-and-ulids/)
* [ulid](https://github.com/ulid/spec)
