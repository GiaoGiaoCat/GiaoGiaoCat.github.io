---
layout: post
title:  "Go 中的闭包"
date:   2019-05-04 08:00:00

categories: go
tags: functional_programming
author: "Victor"
---

## 匿名函数

* 不希望给函数起名字的时候，可以使用匿名函数，例如：`func(x, y int) int { return x + y }`。
* 这样的函数不能独立存在，所以通常都会赋值给某个变量，也就是把函数地址保存到变量中。`fplus := func(x, y int) int { return x + y }`，然后通过变量名对函数进行调用 `fplus(3,4)`。
* 也可以直接对匿名函数进行调用 `func(x, y int) int { return x + y }(3, 4)`

```go
package main

import "fmt"

func main() {
  f()
}
func f() {
  for i := 0; i < 4; i++ {
    g := func(i int) { fmt.Printf("%d ", i) } // 此例子中只是为了演示匿名函数可分配不同的内存地址，在现实开发中，不应该把该部分信息放置到循环中。
    g(i)
    fmt.Printf(" - g is of type %T and has value %v\n", g, g)
  }
}
```

匿名函数也可以接受或不接受参数。

```go
func (u string) {
    fmt.Println(u)
    // ...
}(v)
```

### defer 语句和匿名函数

关键字 defer 经常配合匿名函数使用，它可以用于改变函数的命名返回值，通常是用于在返回语句之后修改返回的 `error`。

```go
package main

import "fmt"

func f() (ret int) {
  defer func() {
    ret++
  }()
  return 1
}
func main() {
  fmt.Println(f())
}
```

变量 ret 的值为 2，因为 `ret++` 是在执行 `return 1` 语句后发生的。

## 闭包

**闭包是由函数和与其相关的引用环境组合而成的实体。** 这一点切忌，基本上这一点搞明白了，就不会用错。

```go
```

1. 闭包对外函数的变量的修改，是对变量的引用
2. 变量被引用后，它所在的函数结束，这变量也不会马上被销毁
  * **闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。**
3. 在闭包中使用到的变量可以是在闭包函数体内声明的，也可以是在外部函数声明的。
4. 一个返回值为另一个函数的函数可以被称之为工厂函数，这在需要创建一系列相似的函数的时候非常有用：书写一个工厂函数而不是针对每种情况都书写一个函数。
5. 闭包会发生变量逃逸，原因如 2

### 如何构造闭包

1. 被嵌套的函数引用到非本函数的外部变量，而且这外部变量不是 *全局变量*。
2. 嵌套的函数被独立了出来（被父函数返回或赋值，变成了独立的个体），而被引用的变量所在的父函数已结束。

### 应用实例

将函数作为返回值是闭包的主要应用方式。

#### 例1 闭包的范式

```go
package main

import "fmt"

func main() {
  // make a special Adder function, a gets value 2:
  TwoAdder := Adder(2)
  fmt.Printf("The result is: %v\n", TwoAdder(3))
}

func Adder(a int) func(b int) int {
  return func(b int) int {
    return a + b
  }
}
```

上面代码的另一种实现。

```go
package main

import "fmt"

func main() {
  var f = Adder()
  fmt.Print(f(1), " - ")
  fmt.Print(f(20), " - ")
  fmt.Print(f(300))
}

func Adder() func(int) int {
  // 在多次调用中，变量 x 的值是被保留的，这是使用闭包的一种典型范式。
  var x int
  return func(delta int) int {
    x += delta
    return x
  }
}
```

#### 例2 斐波那契数列

递归版。

```go
func main() {
  result := 0
  for i := 0; i <= 10; i++ {
    result = fibonacci(i)
    fmt.Printf("fibonacci(%d) is: %d\n", i, result)
  }
}

func fibonacci(n int) (res int) {
  if n <= 1 {
    res = 1
  } else {
    res = fibonacci(n-1) + fibonacci(n-2)
  }
  return
}
```

闭包版。

```go
// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
  x := 0
  y := 1
  return func() int {
    x,y = y,x+y
    return x
  }
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

#### 例3 工厂函数

```go
func MakeAddSuffix(suffix string) func(string) string {
  return func(name string) string {
    if !strings.HasSuffix(name, suffix) {
      return name + suffix
    }
    return name
  }
}
addBmp := MakeAddSuffix(".bmp")
```

#### 例4 一道面试题

想想为什么最后输出的是 3 而不是 2。

```go
var dummy [3]int
var f func()
for i := 0; i < len(dummy); {
	f = func() {
		println(i)
	}
	i++
}
f() // 3
```

## 相关连接

* [闭包](https://wiki.jikexueyuan.com/project/the-way-to-go/06.8.html)
* [Golang 中关于闭包的坑](https://www.jianshu.com/p/fa21e6fada70)
* [Go 语言闭包详解](https://juejin.im/post/5c850d035188257ec629e73e)
