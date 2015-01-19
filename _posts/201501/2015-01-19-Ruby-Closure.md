---
layout: post
title:  "Ruby 闭包"
date:   2015-01-19 21:00:00
categories: ruby
tags: functional_programming
author: "Victor"
---

## 概念

* 你可以像传递对象那样传递闭包, 也可以以后调用。
* 当闭包创建的时候, 它记住了该环境下所有的变量的值。被调用时, 它可以访问这些变量, 甚至这些变量已处于环境的外部或其值已经改变。
* Ruby 通过 ``proc`` 和 ``lambda`` 来实现闭包。

## 例子

```ruby
class SomeClass
  def initialize(value1)
    @value1 = value1
  end

  def value_printer(value2)
    # 创建并返回闭包
    lambda { puts "Value1: #{@value1}, Value2: #{value2}" }
  end

end

def caller(some_closure)
  some_closure.call
end

some_class = SomeClass.new(5)
# 闭包赋值给一个变量
printer = some_class.value_printer("some value")
# 将闭包当做参数传给另外一个函数
caller(printer)

#=> Value1: 5, Value2: some value
```

那么闭包是如何保存内部变量的呢？

1. 闭包创建了它所需要的所有变量的一个备份, 因此是这些副本随着闭包传递。
2. 闭包延长了它所需要的所有变量的生命周期, 没有复制变量, 而是保留了它们的引用, 而且变量本身不可以被垃圾回收器回收掉。

```ruby
class SomeClass
  def initialize(value1)
    @value1 = value1
  end

  def value_incrementer
    lambda { @value1 += 1 }
  end


  def value_printer
    lambda { puts "value: #{ @value1 }"}
  end
end

some_class = SomeClass.new(2)
incrementer_closure = some_class.value_incrementer
printer_closure = some_class.value_printer
3.times do
  incrementer_closure.call
  printer_closure.call
end

#=>
value: 3
value: 4
value: 5
```

结论：Ruby 使用的是延长变量的生命周期的办法。每一次迭代2个闭包操作的都是同一个变量, 因为值在增长。

## 作用

1. 使用闭包来持久化一些状态。
2. 写出更加简明的代码而且还利用了命令式的风格。
