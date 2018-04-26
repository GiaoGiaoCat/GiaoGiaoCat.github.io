---
layout: post
title:  "面向对象设计实践指南 6 - 通过继承获得行为"
date:   2015-01-11 16:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

精心设计的父类易于扩展出新的子类，即使对这个应用程序不怎么了解的程序员也没问题。

## 理解经典的继承

继承解决了多种相关类型共享大量公共行为，但在某些层面又有所差异的问题。它允许你将共享的代码隔离起来，并在抽象类里实现公共的算法，同时又提供了一种允许子类提供特殊化的结构。

继承的核心是一种用于实现 **消息自动委托** 的机制。它为无法理解的消息定义了一条转发的路径。

当你需要大量特殊化某个稳定、公共的抽象结构时，继承是一种成本极低的解决方案。

## 弄清使用继承的地方

当某个预先存在的具体类包含你所需要的大部分行为时，多数时候我们会直接往这个类里添加代码来解决新问题。这种模式表面上能带来好处，但实际上却是有害的。

下面是一段错误的示例：

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

## 误用继承

下面又是一个错误的例子：

```ruby
class MountainBike < Bicycle
  attr_reader :front_shock, :rear_shock

  def initialize(args)
    @front_shock = args[:front_shock]
    @rear_shock = args[:rear_shock]
    super(args)
  end

  #...
end
```

因为 Bicycle 是一个具体的类，它不是为了子类而编写的。当 MountainBike 继承 Bicycle 的时候，会继承它既不要想，也不需要的行为。

**问题起始于 Bicycle 类的名字，让人有了误解。**

## 找出抽象

子类是其父类的特殊化。使用继承，有两件事必须始终确保正确：

1. 正在建模的对象必须有真正的 “一般化 - 特殊化” 关系
2. 使用正确的编码技术

Bicycle 代码了一个抽象类，你不能向它发送 `new` 消息（当然在 Ruby 中并不限制你这样做）。

创建只有一个子类的抽象父类几乎没有什么意义。比较好的时机是当出现三个相似类的时候再去抽象出一个父类。

```ruby
class Bicycle
  attr_reader :size

  def initialize(args = {})
    @size = args[:size]
  end
end

class RoadBike < Bicycle
  attr_reader :tape_color

  def initialize(args)
    @tape_color = args[:tape_color]
    super(args)
  end
end
```

**抽取父类的正确方法应该是，先将方法全部放到子类然后再根据需要移动到父类中。**

这个移动方法的顺序很重要，因为这种 “提升” 失败很容易发现和修复。而如果反着来，先把方法放在父类再根据需要移动到子类中，你可能会不小心在父类中遗留下具体的行为（其它子类并不需要的行为）。

### 使用模板方法

在父类定义好基本结构，并通过发送消息来获取子类特有的实现，这种技术被称作 “模板方法模式”。

```ruby
class Bicycle
  attr_reader :size, :chain, :tire_size

  def initialize(args = {})
    @size = args[:size]
    @chain = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size
  end

  def default_chain
    "10-speed" # 公共的默认值
  end

  def default_tire_size
    raise NotImplementError, "This #{self.class} cannot respond to:"
  end
end

class RoadBike < Bicycle
  def default_tire_size
    "23" # 子类的默认值
  end
end
```

任何使用模板方法模式的类都必须为它所发送的每一条消息提供一个实现。即便这个实现只是为了让其产生有用的错误信息，对模板方法的要求进行记录。

## 管理父类与子类的关系

### 理解耦合

当知道与其他类相关的事情时，便会创建依赖关系。并且，这些依赖关系会将相关对象耦合在一起。在使用继承这一技术时，依赖关系通常都是由子类发送的 `super` 消息所建立的。

任何程序员都可能忘记发送 `super` 消息，从而导致代码出现错误。而这种错误会在远离其起因很远的某个时间点和地点暴露出来，从而使得它们难以调试。

当某个子类发送 `super` 消息时，它实际上是在表明自己知道那个算法，即它依赖于这个知识。如果被依赖的知识发生变化，那么这些子类都可能会遭到破坏。

```ruby
class RoadBike
  attr_reader :tape_color

  def initialize(args = {})
    @tape_color = tape_color
    super(args)
  end
end
```

### 使用钩子消息解耦子类

父类发送钩子消息，该消息是专门为子类设立的地方，方便它们通过实现相匹配的方法来提交信息。这种策略将算法从子类里移除，并将控制权返回给了父类。

**新的子类只需要实现这些模板方法即可。**

好处：

* 子类不需要了解父类的算法，不用发送 `super` 消息，仍然可以提供特殊化内容
* 降低耦合，任何人都可以创建子类而不用了解整个应用程序

```ruby
class Bicycle
  def initialize(args = {})
    @size = args[:size]
    @chain = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size

    post_initialize(args)
  end

  def spares
    { tire_size: tire_size, chain: chain }.merge(local_spares)
  end

  def default_tire_size
    raise NotImplementError
  end

  # 子类可改写
  def post_initialize(args)
    nil
  end

  def local_spares

  end

  def default_chain
    '10-speed'
  end
end
```

```ruby
class RoadBike < Bicycle
  attr_reader :tape_color

  def post_initialize(args)
    @tape_color = args[:tape_color]
  end

  def local_spares
    { tape_color: tape_color }
  end

  def default_tire_size
      '23'
  end
end
```

## 小结

* 创建抽象父类的最好方法是，从具体的子类里提取代码。
* 等到有三种高度相关的类时再抽取父类。
* 父类可以使用模板方法 + 钩子消息模式，让子类提供特殊化并解耦。
* 任何使用模板方法模式的类都必须为它所发送的每一条消息提供一个实现。
