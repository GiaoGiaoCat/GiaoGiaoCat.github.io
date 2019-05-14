---
layout: post
title:  "Go 的并发"
date:   2019-05-13 08:00:00

categories: go
tags: concurrency
author: "Victor"
---

## 基础

Go 语言秉承 CSP 并发模式通过 goroutine 和 channel 实现并发。goroutine 类似于线程但是由 Go 语言的运行时调度完成，而线程是由操作系统调度完成。

* Go 程序从 main 包的 `main()` 函数开始，在程序启动时，就会为 `main()` 函数创建一个默认的 goroutine。
* 使用 go 关键字创建 goroutine 时，被调用函数的返回值会被忽略。
* 使用匿名函数创建 goroutine 时，需要加上匿名函数的调用参数。
* 所有 goroutine 会在 `main()` 函数结束时一同结束，而终止 goroutine 的最好方法就是自然返回 goroutine 对应的函数。

```go
go func() {
  var times int
  for {
    times++
    // do something
    time.Sleep(time.Second)
  }
}()
```

### 并发和并行

* 并发 concurrency 把任务在不同的时间点交给处理器进行处理。在同一时间点，任务并不会同时运行。
* 并行 parallelism 把每一个任务分配给每一个处理器独立完成。在同一时间点，任务一定是同时运行。

两个概念的区别是，任务是否同时执行。Go 在 GOMAXPROCS 数量与任务数量相等时，可以做到并行执行，但一般情况下都是并发执行。

## 通道

channel 是一种特殊的类型。在任何时候，同时只能有一个 goroutine 访问通道进行发送和获取数据。goroutine 间通过 channel 就能通信。channel 遵循先入先出原则。

通道和切片一样需要一个类型来限定内部传输的数据类型。chan 类型的空值是 nil，声明后需要配合 make 后才能使用。

* 使用通道发送数据，将持续阻塞直到数据被接收。
* 使用通道接收数据，将持续阻塞直到发送方发送数据。

通道的非阻塞接受数据模式，因为造成较高的 CPU 占用，因此使用非常少。

```go
func main() {
	ch := make(chan int)
	go func() {
		fmt.Println("start goroutine")
		ch <- 0
		fmt.Println("exit goroutine")
	}()

	fmt.Println("wait goroutine")
	<- ch
	fmt.Println("all done")
}
```

循环取出数据：

```go
func main() {
	ch := make(chan int)
	go func() {
		for i := 3;	i >= 0;	i-- {
			ch <- i
			time.Sleep(time.Second)
		}
	}()

	for data := range ch {
		fmt.Println(data)
		// 当遇到数据 0 时，退出接收循环
		if data == 0 {
			break
		}
	}
}
```

通过信道阻塞的方式实现并发处理，是 channel 的典型范式之一，这里有个小技巧可以通过 0 来结束循环。

```go
func printer(c chan int) {
	// 开始无限循环等待数据
	for {
		// 从 channel 中获取一个数据
		data := <- c
		// 将 0 视为数据结束
		if data == 0 {
			break
		}
		fmt.Println(data)
	}
	// 通知 main 已经结束循环
	c <- 0
}

func main() {
	c := make(chan int)
	// 并发执行 
	go printer(c)

	for i := 1; i <= 10; i++ {
		// 将数据通过 channel 投送给 
		c <- i
	}
	// 通知并发的 printer 结束循环
	c <- 0
	// 等待 printer 结束
	<- c
}
```

### 通道的多路复用

多路服用通常表示在一个信道上传输多路信号和数据的过程和技术。

Go 语言提供 select 关键字，可以同时响应多个通道的操作。select 的每个 case 都会对应一个通道的收发过程。当收发完成时，就会触发 case 中响应的语句。多个操作在每次 select 中随机挑选一个进行响应。

## 处理并发的方式

单纯地函数并发是没有意义的。函数与函数间需要交换数据才能提现并发执行函数的意义。如果使用共享内存进行数据交换，会发生竟态问题。为了保证数据交换的正确性，必须使用互斥量进行加锁，这也就会有性能问题。

并发，就会出现竞争问题，下面是几种常见的处理方式：

* 加锁。所有的参与者竞争同一个锁，持有锁的参与者可以对资源进行操作。如果资源满足读和写可以分离的条件，可以使用读写锁来 提高读的并发数量。
* 单线程顺序执行。Redis的处理IO的线程就是这样操作的，因为Redis属于I/O密集型应用而不是CPU密集型应用，所以虽然是单线程， CPU仍然不会是性能瓶颈，单线程的好处就在于处理请求的时候，一个一个按顺序来，所以无需加锁，可以简化实现。
* 原子操作。参考 Go 中的 `sync/atomic` 包。使用加锁的方式，在持有锁的时间内，可以进行多个操作，而原则操作通常是一条语句， 例如把某个整数加上n，例如当某个数等于m时，复制为n等等。

## 原文链接

* [处理并发的方式](https://jiajunhuang.com/articles/2018_11_07-concurrency.md.html)
