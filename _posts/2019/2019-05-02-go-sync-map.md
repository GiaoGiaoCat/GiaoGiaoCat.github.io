---
layout: post
title:  "并发环境使用的 sync.Map"
date:   2019-05-02 18:00:00

categories: go
tags: concurrency
author: "Victor"
---

## map 在并发下遇到的问题

Go 语言中的 map 在并发情况下，只读是线程安全的，同时读写线程不安全。

```go
func main() {
  m := make(map[int]int)

  // 开启一段并发代码
  go func() {
    for {
      // 不停对 map 进行写入
      m[1] = 1
    }
  }()

  // 开启一段并发代码
  go func() {
    for {
      // 不停对 map 进行读取
      _ = m[1]
    }
  }()

  // 无限循环，让并发程序在后台执行
  for {
  }
}
```

感谢 Go 足够聪明，运行这段代码会直接报错 `fatal error: concurrent map read and map writ`。两个并发函数不断地对 map 进行读和写发生了竞态问题。map 内部会对这种并发操作进行检查并提前发现。

## sync.Map 的特性

需要并发读写时，一般的做法是加锁，但这样性能不高，且要自己维护锁。推荐使用 sync.Map

* 无须初始化，直接声明即可。
* 不能使用 map 的方法来操作，Store 表示存储，Load 表示获取，Delete 表示删除。
* 使用 Range 配合一个回调函数进行遍，通过回调函数返回内部遍历出来的值。Range 的返回值为 bool 其代表是否要继续迭代遍历。

```go
func main() {
  var scence sync.Map
  scence.Store("greece", 97)
  scence.Store("london", 100)
  scence.Store("egypt", 200)

  fmt.Println(scence.Load("london"))

  scence.Delete("egypt")

  scence.Range(func(k, v interface{}) bool {
    fmt.Println("iterate:", k, v)
    return true
  })
}
```
