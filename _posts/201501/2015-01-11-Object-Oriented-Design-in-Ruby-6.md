---
layout: post
title:  "面向对象设计实践指南 6 - 通过继承获得行为"
date:   2015-01-11 16:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

## 理解经典的继承

继承的核心是一种用于实现 **消息自动委托** 的机制。它为无法理解的消息定义了一条转发的路径。

## 弄清使用继承的地方

当某个预先存在的具体类包含你所需要的大部分行为时，采用往类里添加代码的方式解决问题是非常吸引人的，因为它简单又直接。这种模式表面上能带来好处，但实际上却是有害的。

```ruby
class Bicycle
  attr_reader :style, :size, :tape_color, :front_shock, :rear_shock

  def initialize(args)
    @style = args[:style]
    @size = args[:size]
    @tape_color = args[:tape_color]
    @front_shock = args[:front_shock]
    @rear_shock = args[:rear_shock]
  end

  def spares
    if style == :road
      { chain: '10-speed', tire_size: '23', tape_color: tape_color }
    else
      { chain: '10-speed', tire_size: '2.1', rear_shock: rear_shock }
    end
  end
end
```

Bicycle 类现在有了多种职责。`spares` 包含了一个 if 语句，它会检查 self 的某属性，以此确定向 self 发送什么样的消息。通过上一章我们知道，这个模式与鸭子类型有关：**通过检查对象的类来确定向该对象发送什么样的消息。**

继承要解决的问题：类型之间共享一些行为，只在某些层面有所不同。

消息通过继承在类之间进行转发。而鸭子类型是跨类的，所以他们无法使用继承来共享共同的行为。鸭子类型通过 Ruby 模块实现代码共享。

![](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/06-01.png)
