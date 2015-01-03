---
layout: post
title:  "面向对象设计实践指南 3 - 管理依赖关系"
date:   2015-01-03 23:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

因为精心设计的对象都具有单一职责，因此它们在本质上要求通过合作来完成复杂的任务。这强大而危险。为实现合作，一个对象必须知道其它对象的某些情况。这便形成了一种依赖关系。

## 理解依赖关系

依赖关系意味着，一个类要因为另一个类的变化而变化。

### 认识依赖关系

当某个对象知道了以下内容时，它便会形成一种依赖关系：

* 另一个类的名字
* 消息的名字
* 消息所要求的参数
* 参数的顺序

### 对象间的耦合

依赖关系会将两个类耦合在一起。或者说每一个耦合都会创建一种依赖关系。当有两个及以上的对象紧密耦合在一起时，它们便会表现的像是一个整体，不可能只重用当中的某一个。更改会迫使大家一起更改。

### 其它依赖关系

有一种极具破坏性的依赖关系，即一个对象知道另一个对象，而那个对象又知道另外一个对象。为了获得遥远对象里的行为，许多消息会被串联在一起。对中间对象的任何更改都会影响整个消息链。

## 编写松耦合的代码

### 注入依赖关系

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def gear_inches
    radio * Wheel.new(rim, tire).diameter
  end
end

Gear.new(52, 11, 26, 1.5).gear_inches
```

#### 缺点

* 通过类名字引用另一个类会创建一个依赖关系。如果类 Wheel 的名字发生变化，那么 Gear 的 ```gear_inches``` 的方法必须跟着变化。
* 它决绝与其它类合作，哪怕这个类有 ```rim``` 和 ```tire``` 也不行。

而实际上，Gear 需要访问一个可以响应 ```diameter``` 的对象（鸭子类型）。

将 Wheel 实例的创建移动到 Gear 的外面能将两个类解耦出来。 Gear 现在可以与任何实现了 ```diameter``` 的对象进行合作。

这种技术被成为 **依赖注入(dependency injection)**。

#### 优点

* Gear 之前对 Wheel 类以及初始化参数的类型和顺序有明确的依赖，现在减少到仅对 ```diameter``` 方法的单一依赖。
