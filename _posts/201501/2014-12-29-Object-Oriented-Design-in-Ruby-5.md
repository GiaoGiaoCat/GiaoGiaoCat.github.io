---
layout: post
title:  "面向对象设计实践指南 5 - 使用鸭子类型技术降低成本"
date:   2015-01-07 20:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

面向对象设计的目的在于要减少变化所带来的成本。既然我们知道了消息是应用程序的设计中心，而且致力于构建出严格定义的公共接口，那么可以结合一下这两种技术，进一步降低成本，也就是鸭子类型。

鸭子类型指的是不会绑定到任何特定类的公共接口。这种跨类接口能为应用程序带来巨大的灵活性，所采用的方式是利用更加宽容的 **消息依赖** 取代昂贵的 **类依赖**。

## 理解鸭子类型

### 什么是类型

类型就是描述变量内容的分类。

过程语言提供了少量固定的类型，通常用来描述各种类型的数据。而变量类型，让应用程序假设：数字可以用于数学表达式、字符串可以连接、数组可以索引。

### Ruby 中的类型

在 Ruby 中，对某个对象的行为期望是以信赖其公共接口的形式表现出来。如果知道一个对象的类型，就知道这个对象可以响应哪些消息。对象的使用者不需要关心它的类。类只是对象获得公共接口的一种方式。

**我们要关心的是对象具体做什么，而不是这个对象是什么。**

### 鸭子类型概述

```ruby
class Trip
  attr_reader :bicycles, :customers, :vehicle

  # 这个 mechanic 参数可以使任何类
  def prepare(mechanic)
    mechanic.prepare_bicycles(bicycles)
  end
end

class Mechanic
  def prepare_bicycles(bicycles)
    bicycles.each {|bicycle| prepare_bicycle(bicycle)}
  end

  def prepare_bicycle(bicycle)
    # do somthing
  end
end
```

`prepare` 方法对 Mechanic 类没有显示的依赖关系。实际上，它依赖于接收到的那个可以响应 `prepare_bicycles` 的对象。

流程图如下：

```
SomeObject -> Trip: perpare(mechanic)
activate SomeObject
activate Trip

Trip -> Mechanic: prepare_bicyles(bicycles)
activate Mechanic
Mechanic -> Trip
deactivate Mechanic

Trip -> SomeObject
deactivate Trip
deactivate SomeObject
```

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/Object-Oriented-Design-in-Ruby/05-01.png)

### 让问题复杂些

当需求发生变化之后，我们出现了如下的代码：

```ruby
class Trip
  attr_reader :bicycles, :customers, :vehicle

  def prepare(prepares)
    prepares.each do |preparer|
      case preparer
       when Mechanic
         preparer.prepare_bicycles(bicycles)
       when TripCoordinator
         preparer.buy_food(customers)
       when Driver
         preparer.gas_up(vehicle)
         preparer.fil_water_tank(vehicle)
       end
    end
  end
end

class Mechanic
  def prepare_bicycles(bicycles)
  end
end

class TripCoordinator
  def buy_food(customers)
  end
end

class Driver
  def gas_up(vehicle)
  end
  def fil_water_tank(vehicle)
  end
end
```

当你基于类的角度来解决问题的时候，就会发现自己要去处理某些对象，它们并不理解你正发送的消息，这时你会试着去找出这些新对象所能理解的信息。去查看那些类的公共接口，然后把那些行为添加到自己的方法内。

你从 **要什么** 改成了通知类们要 **如何做**。这种编码风格会自我传播，其他人会顺着你的路走下去，增加新的 `when` 分支。最终，全部重写比更改更容易。

这时候你可以借用时序图来整理设计思路，**时序图应该永远比它们所表示的代码简单**，可是你却画出了超级复杂的时序图。

### 发现鸭子类型

Trip 的 prepare 方法参数的类没有预定的期望，它希望每个参数都是准备员。我们发现了一个抽象的的类 Prepare。Mechanic、TripCoordinator 和 Driver 都应该表现的像 Prepare 一样，也就是说它们都该实现 `prepare_trip` 方法。

```ruby
class Trip
  attr_reader :bicycles, :customers, :vehicle

  def prepare(prepares)
    prepares.each {|preparer| preparer.prepare_trip(self)}
  end
end

class Mechanic
  def prepare_trip(trip)
    trip.bicycles.each {|bicycle| prepare_bicycle(bicycle)}
  end
end
class TripCoordinator
  def prepare_trip(trip)
    buy_food(trip.customers)
  end
end
class Driver
  def prepare_trip(trip)
    vehicle = trip.vehicle
    gas_up(vehicle)
    fil_water_tank(vehicle)
  end
end
```

### 鸭子类型的好处

最初那个例子里 `prepare` 方法依赖于一个具体类，最后的例子里 `prepare` 依赖于一个鸭子类型。鸭子类型更加抽象，对理解的要求稍微高一些，但是换来的是扩展的便捷。

能容忍某个对象的类存在歧义的能力，是充满自行的设计师所具备的品质。一旦你开始认为对象是通过它们的行为，而非通过它们的类进行定义的，那么你便进入一个新的表现灵活的设计领域。

## 编写依赖于鸭子类型的代码

### 识别出隐藏的鸭子类型

有几种常见的编码模式表示出有隐藏的鸭子类型存在：

* 选择类的 `case` 语句
* `kinf_of?` 和 `is_a?`
* `responds_to?` 略微减小了依赖关系的数量，但仍然与类耦合在一起。因为它期望的仍然是每一个可以响应独特方法的类。

### 信任你的鸭子类型

`case` `kinf_of?` `is_a?` `responds_to?` 这些代码实际上都是说：我知道你是谁，因为我知道你要做什么。这意味着缺乏对合作对象的信任。

这种编码风格表明了一种迹象：你漏掉了某个公共接口还未被发现的对象。漏掉的那个对象是一种鸭子类型，而非具体的类。因为这些代码实际上都在说：我知道你是谁，我知道你要做什么。

### 记录好鸭子类型

当创建鸭子类型时，必须要同时做好记录和测试它们的公共接口。幸运的是，好的测试就是最好的记录。因此，只要编写测试就行。

### 在鸭子类型之间共享代码

在实现鸭子类型的时候，经常需要共享某个公共的行为。第7章会讲解这些。

### 合理选择鸭子类型

```ruby
def first(*args)
  if args.any?
    if args.first.kind_of(Integer) || (loaded? && !args.first.kind_of?(Hash))
      to_a.first(*args)
    else
      apply_finder_options(args.first).first
    end
  else
    find_first
  end
end
```

这个例子与之前的区别在于那些被检查类的稳定性。它 `first` 依赖于 Ruby 核心类的时候，这种依赖关系是安全的。

## 克服对鸭子类型的恐惧

如果你对静态类型和动态类型的优缺点持怀疑态度，那么可以看本章节。这里不做笔记了。

## 小结

消息是面向对象应用程序的中心，它们会在对象之间沿着公共接口传递。鸭子类型能将公共接口与特定类分离开来，并创建出 **根据其做什么，而非是什么进行定义** 的虚拟类型。

鸭子类型能揭示那些可能还未被发现的底层抽象。依赖这些抽象能降低风险和增加灵活性，使应用程序的维护成本更低，且更易于更改。
