---
layout: post
title:  "面向对象设计实践指南 3 - 管理依赖关系"
date:   2015-01-03 23:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

因为精心设计的对象都具有单一职责，因此它们在本质上要求通过合作来完成复杂的任务。这强大而危险。为实现合作，一个对象必须知道其它对象的某些情况。这便形成了一种依赖关系。

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
    ratio * Wheel.new(rim, tire).diameter
  end

  def ratio
    chainring / cog.to_f
  end
end

class Wheel
  attr_reader :rim, :tire
  def initialize(rim, tire)
    @rim = rim
    @tire = tire
  end

  def diameter
    rim + (tire * 2)
  end
end
```

## 理解依赖关系

依赖关系意味着，一个类要因为另一个类的变化而变化。

### 认识依赖关系

当某个对象知道了以下内容时，它便会形成一种依赖关系：

* 另一个类的名字
* 方法的名字(另外一个类的实例的方法)
* 方法所要求的参数
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
* 它拒绝与其它类合作，哪怕这个类有 ```rim``` 和 ```tire``` 也不行。

而实际上，Gear 需要访问一个可以响应 ```diameter``` 的对象（鸭子类型）。

将 Wheel 实例的创建移动到 Gear 的外面能将两个类解耦出来。 Gear 现在可以与任何实现了 ```diameter``` 的对象进行合作。

这种技术被成为 **依赖注入(dependency injection)**。

#### 优点

* Gear 之前对 Wheel 类以及初始化参数的类型和顺序有明确的依赖，现在减少到仅对 ```diameter``` 方法的单一依赖。

### 隔离依赖关系

最好是将所有不必要的依赖关系都切断，但实际当中很难办到。因此，如果不能移除不必要的依赖关系，就将它们隔离在你的类里。

#### 隔离实例创建

第一种方法是把创建新的 Wheel 实例的操作从 ```gear_inches``` 方法移动到了 Gear 的初始化方法。

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @wheel = Wheel.new(rim, tire)
  end

  def gear_inches
    ratio * @wheel.diameter
  end
end
```

第二种方法将创建新的 Wheel 的操作隔离在自己明确定义的 wheel 方法里。

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire, :wheel
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def gear_inches
    ratio * wheel.diameter
  end

  private
  def wheel
    @wheel ||= Wheel.new(rim, tire)
  end
end
```

以上的两种方法，虽然 Gear 仍然知道的太多，但已经有所改进。它减少了 ```gear_inches``` 里的依赖关系数量，同时公开地暴露了 Gear 对 Wheel 的依赖。在条件允许时，这样的代码更容易重构。

#### 隔离脆弱的外部信息

```ruby
def gear_inches
  # 几行数学运算
  foo = some_intermediate_result * wheel.diameter
  # 几行数学运算
end
```

```wheel.diameter``` 被嵌入在一个复杂的方法里。移除外部依赖关系，并将其封装在自己的某个方法里，可以降低修改 ```gear_inches``` 的机率。

```ruby
def gear_inches
  # 几行数学运算
  foo = some_intermediate_result * diameter
  # 几行数学运算
end

def diameter
  wheel.diameter
end
```

如果 Wheel 更改了 diameter 的名字或方法名，那么对 Gear 的副作用会被限制在一个简单的包裹方法里。

### 移除参数顺序依赖关系

#### 使用散列表初始化参数

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(args)
    @chainring = args[:chainring]
    @cog = args[:cog]
    @wheel = args[:wheel]
  end
end
Gear.new(:chainring => 52, :cog => 11, :wheel => Wheel.new(26, 1.5).gear_inches)
```

虽然不再依赖参数顺序，但是它依赖散列表中的键名。这种更改和依赖是正确的，它还提供了对参数的文档功能。

#### 显式定义默认值

为了设置默认值，我们进行如下修改，请看如何一步一步提高代码的质量：

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(args)
    @chainring = args[:chainring] || 40
    @cog = args[:cog] || 18
    @wheel = args[:wheel]
  end
end
```

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(args)
    @chainring = args.fetch(:chainring, 40)
    @cog = args.fetch(:cog, 18)
    @wheel = args[:wheel]
  end
end
```

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(args)
    args = defaults.merge(args)
    @chainring = args[:chainring]
    @cog = args[:cog]
    @wheel = args[:wheel]
  end

  def defaults
    { :chainring => 40, :cog => 18 }
  end
end
```

#### 隔离多参数初始化操作

假设在你的方法内部需要创建某个类的实例，而你不能修改这类。那么可以创建一个单一方法，将外部接口包裹起来。

下面的示例里面 ```SomeFramework::Gear``` 不属于我的应用程序，它是外部框架。

```ruby
module SomeFramework
  class Gear
    attr_reader :chainring, :cog, :wheel
    def initialize(args)
      @chainring = args[:chainring]
      @cog = args[:cog]
      @wheel = args[:wheel]
    end
  end
end

module GearWrapper
  def self.gear(args)
    SomeFramework::Gear.new(args[:chainring], args[:cog], args[:wheel])
  end
end

GearWrapper.gear(:chainring => 52, :cog => 11, :wheel => Wheel.new(26, 1.5).gear_inches)
```

GearWrapper 是一个 Ruby 模块，它负责创建 SomeFramework::Gear 实例。

这里使用模块的意思是，我们不想创建 GearWrapper 的实例。它只有一个目的，创建其他某个类的实例。这叫做 **工厂(factory)**。

对于某个对象，如果其目的是用于创建其他的对象，那么它就是一个工厂。

## 管理依赖方向

### 翻转依赖关系

```ruby
class Gear
  attr_reader :chainring, :cog
  def initialize(args)
    @chainring = args[:chainring]
    @cog = args[:cog]
  end

  def gear_inches(diameter)
    ratio * diameter
  end

  def ratio
    chainring / cog.to_f
  end
end

class Wheel
  attr_reader :rim, :tire, :gear
  def initialize(rim, tire, chainring, cog)
    @rim = rim
    @tire = tire
    @gear = Gear.new(chainring, cog)
  end

  def diameter
    rim + (tire * 2)
  end

  def gear_inches
    gear.gear_inches(diameter)
  end
end

Wheel.new(26, 1.5, 52, 11).gear_inches
```

这种反转的依赖并无害处，也没啥好处。你可能觉得无关紧要，但依赖方向的选择会带来深远的影响。如果选择正确，代码更容易维护；选择错误，依赖关系逐步成为问题的根源，最终应用程序越来越难维护。

### 选择依赖方向

仅有一条原则，依赖那些变化情况比你所做的更改要少的事物。也就是说尽量依赖变化更少的类。

* 有些类比其他类更容易发生需求变化。
* 具体类比抽象类更容易发生变化。
* 更改拥有许多依赖关系的类会造成广泛的影响。

这里的关键时如何区别具体与抽象。当我们使用 **依赖注入(dependency injection)** 技术之后，Gear 开始依赖于某种抽象的事物，它依赖于能响应 ```diameter``` 消息的对象(鸭子类型)。这就是抽象思想。抽象源自具体类，但与任何特定的实例分离开来。
