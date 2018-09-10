---
layout: post
title:  "Reids 常见面试问题汇总"
date:   2018-09-09 14:00:00

categories: tool
tags: caching
author: "Victor"
---

## 基础概念

Redis 是一个基于内存的高性能 Key-Value 数据库。

**Redis 适合的场景主要局限在较小数据量的高性能操作和运算上。**

### 优点

* 因为是纯内存操作，Redis 的性能非常出色，每秒可以处理超过 10 万次读写操作
* 支持保存多种数据结构，此外单个 value 的最大限制是 1GB，memcached 只能保存 1MB 的数据
* 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
* 可以选择持久化数据

### 缺点

* 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写

## 常见性能问题和解决方案

1. Master 写内存快照，save 命令调度 rdbSave 函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以 Master 最好不要写内存快照。
2. Master AOF 持久化，如果不重写 AOF 文件，这个持久化方式对性能的影响是最小的，但是 AOF 文件会不断增大，AOF 文件过大会影响 Master 重启的恢复速度。Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个 Slave 开启 AOF 备份数据，策略为每秒同步一次。
3. Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的CPU和内存资源，导致服务 load 过高，出现短暂服务暂停现象。
4. Redis 主从复制的性能问题，为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。

## 数据淘汰（回收）策略

该策略存在的意义是保证 Redis 中均为热点数据，Redis 共提供 6 种策略：

1. volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
5. allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
6. no-enviction（驱逐）：禁止驱逐数据

## 线程和并发

* Redis 是单进程单线程的
* 利用队列技术将并发访问变为串行访问，消除了传统数据库串行控制的开销。
* 本身没有锁的概念，对于多个客户端连接不存在竞争
* 部分客户端对 Redis 进行并发访问时发生的连接超时、数据转换错误、阻塞、客户端连接关闭等问题，**这些都是客户端的问题**

### 如何解决并发竞争的问题



## 原文

* [Redis的那些最常见面试问题](http://database.51cto.com/art/201809/582819.htm)
