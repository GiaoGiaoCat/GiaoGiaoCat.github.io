---
layout: post
title:  "面向对象设计实践指南 7 - 使用模块共享角色行为"
date:   2015-01-12 16:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

## 理解角色

有时候需要在不相关的对象之间共享行为。

### 找出角色

前面介绍的 Preparer 就是一个角色。那些实现了 `preparer` 接口的对象扮演了这个角色。而 Preparer 角色的存在表明了还存在有一个平行的 Preparable 角色。该接口包含了 Preparer 希望发送给 Preparable 的所有信息，也就是 `bicycles, customers, vehicle`。

尽管 Preparer 角色有多个演员，但它非常简单，直接通过接口来定义就行。为了扮演这个角色，对象只需要实现自己的 `prepare_trip` 方法。

但是实际项目中我们往往会发现很多复杂的角色，它们不仅需要特定的消息，还包括特定的行为。当某个角色需要共享行为时，我们就面临着组织代码的问题。在理想情况下，代码应该只定义在一个地方，而任何希望表现得像鸭子类型的对象都可以使用它。

在 Ruby 中，这类混入内容都被称为模块 `module`，它允许你一次定义一组命名方法，这些命名方法独立于类，并且可以混入到任何对象。

### 组织职责

在决定是否要创建鸭子类型，并把共享行为变成一个模块之前，必须要知道正确的做法。

以一个具体的旅行调度问题为例，在特定的时间点需要安排旅行，涉及自行车、机械师和摩托车。这些领域对象都有一些自己独特的特征，自行车和摩托车需要保养，机械师需要休息。具体的要求是：在两次旅行之间，自行车的交付时间为至少一天，摩托车交付时间最少三天，机械师的交付时间是四天。

假设有一个 Schedule 类，实现了如下三种方法：

```ruby
scheduled?(target, starting, ending)
add(target, starting, ending)
remove(target, starting, ending)
```

target 的实例包含 Bicycle, Vehicle, Mechanic。Schedule 会检查传入的 target 的类，以决定 lead time 来计算 starting。

![](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-05.png)

通过上图可以看到，我们在这一过程中 Schedule 会检查类，以便知道要使用什么值。这说明 Schedule 知道的太多，这种知识不应该属于 Schedule，而应该属于 Schedule 正检查的那个类。

### 删除不必要的依赖关系

#### 发现 Schedulable 鸭子类型

Schedule 会检查类型，以确定在变量里要放什么值，这一事实表明这个变量名应该成为一条消息，进而发送给每一个传入的对象。很好，我们发现了 Schedulable 鸭子类型。

我们用一张新的时序图来阐述这一过程。从 `schedulable?` 方法里移除了对类的检查，并调整这个方法，以便将 `lead_days` 消息发送给传入的参数 `target`。

![](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-06.png)

Schedule 显然并不关心 `target` 的类，它只希望这个对象能应答某个特性的消息 `lead_days`。这种基于消息的期望，脱离了对类的依赖，并且暴露了这样一个角色：它被所有的 target 扮演，并且在时序图里清晰可见。

#### 让对象自己说话

发现和使用鸭子类型让代码得到了改善，它移除了 Schedule 对特定类名的依赖，从而让程序更灵活和更容易维护。不过，上面的时序图仍然包含了不必要的依赖。

暂时忘记上面的例子，举一个更极端的例子。假设有一个 StringUtils 类，它实现了管理字符串的方法，你可以发送 `StringUtils.empty?(some_string)` 来询问某个字符串是否为空。

如果你已经有一定关于 Ruby 的经验，那么马上就能发现这个想法很傻。字符串是对象，它有自己的行为，会自我管理。让第三方来管理字符串的行为会增加不必要的依赖。

这个例子说明了一个 Ruby 中的一个重要道理 **对象应该自我管理**。 它们应该包含自己的行为。如果你对 B 对象感兴趣，那么你不应该被迫知道 A 对象。即使你只是用 A 来查找关于 B 对象的事情，也不应该这样做。

所以上面的时序图违反了这一原则，询问 Schedule 的某个目标是否可调度，有点像在问 StringUtils 类，某个字符串是否为空。

就像字符串本身应该有 `empty?` 方法一样，那些 target 也应该响应 `schedulable?` 消息，这个消息应该被添加到 Schedulable 角色的接口里。

### 编写具体代码

虽然我们发现了 `schedulable?` 但是还不知道它应该写在哪里，代码包含什么内容。但是我们记得前一章提到的一个技术 **抽取父类的正确方法应该是，先将方法全部放到子类然后再根据需要移动到父类中。**

好，先把 `schedulable?` 放在 Bicycle 里再说。

![](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-07.png)

```ruby
class Bicycle
  attr_reader :schedule, :size, :chain, :tire_size

  # 注入 Schedule 并提供默认值
  def initialize(args = {})
    @schedule = args[:schedule] || Schedule.new
    #...
  end

  # 如果这辆车在范围时间内可用则返回 true
  def schedulable?(start_date, end_date)
    !schedulable?(start_date - lead_days, end_date)
  end

  # Return the schedule's answer.
  def scheduled?
    schedule.scheduled?(self, start_date, end_date)
  end

  # Return the number of lead_days before a Bicycle can be scheduled.
  def lead_days
    1
  end
end
```

```ruby
class Schedule
  def scheduled?(schedulable, start_date, end_date)
    puts "This #{schedulable.class} is not scheduled between #{start_date} and #{end_date}"
    false
  end
end
```

Bicycle 可以正确调整开始日期，并将 Schedule 是谁以及 Schedule 做了什么的知识，隐藏在了 Bicycle 里面。

### 提取抽象

我们已经知道了 `schedulable?` 方法应该做什，现在就该抽取 module 了。

```ruby
module Schedulable
  attr_writer :schedule

  def schedule
    @schedule ||= ::Schedule.new
  end

  def schedulable?(start_date, end_date)
    !schedulable?(startt_date - lead_days, end_date)
  end

  def scheduled?(start_date, end_date)
    schedule.scheduled?(self, start_date, end_date)
  end

  # Could be implemented by who mixins me.
  def lead_days
    0
  end
end
```

```ruby
class Bicycle
  include Schedulable

  def lead_days
    1
  end
end
```

消息模式从向 Bicycle 发送 `schedulable?` 消息，变成了向 Schedulable 发送 `schedulable?`。

![](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-08.png)

经典继承与通过模块实现代码共享的差别是： “是一个” 与 “像一个” 的差别，不同的选择带来不同的后果。但这两种编码技术很相似，而且它们都依赖消息委托。

### 继承角色行为

模块是个很强大的工具，你可以编写出难以理解、调试或扩展的代码。但你的任务不是避开这些技术，而是学着用正确的理由、在正确的地方、以正确的方式使用它们。

首先要做到的是正确的编写可继承的代码。

## 编写可继承的代码

代码的实用性和可维护性与代码质量成正比。

### 识别出反模式

1. 使用类似 type 或 category 这类名字的变量来确定发送什么消息给 self
  * 缺点：每次有新的类型加入时，必须更改代码
  * 建议：使用继承来重构
  * 优势：可以通过添加新的子类来创建新的子类型，在不改变现有代码的基础上扩展这个层次结构
2. 当某个发送对象要检查接收对象的类，以确定所发送的消息时
  * 缺点：每次引入新的接收类，必须更改代码
  * 建议：使用鸭子对象来重构
  * 优势：原来的发送者不用再担心任何事，每个接收者都可以理解这条公共消息

### 坚持抽象

抽象父类里的所有代码都应该适用于每一个继承它的类，父类不应该包含只适用于部分子类的代码，这个限制也适用于模块。

### 重视契约

我们在编写子类时应该遵守里氏替换原则 LSP，既接受相同类型的输入，以及返回同样类型的输出。也就是说子类可以在任何情况下替换其父类。

里氏替换原则 LSP 的解释：对于一个健全的类型系统，其子类型必须能够替换它们的父类型。

* 该原则在数学上被表述为：子类型应该能够替换它的父类型
* 在 Ruby 里它表示的是：对象应该言行一致

### 使用模板方法模式

用于创建可继承代码的基本编码技术是模板方法模式。

### 预先将类解耦

尽量避免编写需要继承者发送 super 消息的代码，可以通过钩子消息让子类参与进来，同时还可免除它们要知道抽象算法的职责。

### 创建浅层结构

钩子方法的局限性在于它仅适用于创建浅层结构。

每一个层次结构都可以被看作是一座金字塔，它有深度和广度。深度指它和顶部之间所有父类的数量，宽度指它的直接子类数量。层次结构的形状是由其整体的广度和深度定义，并且正是这个形状确定了它的易用性、易维护性和易扩展性。

下面依次列出：浅窄、浅宽、深窄、深宽 4种不同的层次结构。

![浅窄](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-01.png)
![浅宽](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-02.png)

![深窄](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-03.png)
![深宽](http://wjp2013.github.io/assets/images/pictures/Object-Oriented-Design-in-Ruby/07-04.png)

深层次结构的问题在于，它定义了一条很长的方法查找链，内建了很大的依赖关系集合，而且程序员往往只对顶端和底部的类比较熟悉，会加大引入错误的几率。

## 小结

* 当扮演公共角色的对象需要共享行为时，可以通过 Ruby 的模块实现。
* 模块里定义的代码可以添加到任何对象，其中包括类实例、类本身或者另一个模块。
* 编写子类时应该遵守里氏替换原则 LSP。
* 模块应该使用模板方法模式，从而让哪些包含它们的类型可以提供自己特殊化的内容。
* 模块也应该实现钩子方法，以避免那些混入了模块的类型必须发送 `super` 消息才能知道算法。
* 代码应该是浅层结构，以方便理解和减少错误。
