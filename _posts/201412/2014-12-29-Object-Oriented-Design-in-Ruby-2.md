---
layout: post
title:  "面向对象设计实践指南 2 - 设计具有单一职责的类"
date:   2014-12-29 17:30:00
categories: ruby
tags: learningnote refactoring
author: "Victor"
---

## 决定类的内容

面向对象系统的基础和核心是消息(message)，但最明显的组织结构是类(class)。

### 将方法组织成类

* 你所创建的类会永久的影响你对这个应用程序的思考。
* 尽管将方法正确分组成类很重要，但在项目的早期由于所了解的需求太少，所以不可能将它处理得很正确。
* 设计是保留可变性的艺术，而非达到完美性的行为。

### 组织代码以便于更改

#### 易于更改

* 更改不会产生意想不到的副作用
* 需求的轻微变化对代码的更改要求也相应较小
* 现有的代码易于重用
* 最简单的更改方式是添加其自身也易于更改的代码

#### 编写出的代码应该具有以下几个特点(TRUE 原则)

* 透明性 Transparent: 在当前类和依赖于当前类的其它类中，更改所产生的后果显而易见
* 合理性 Reasonable: 更改的成本和效益成正比
* 可用性 Usable: 现有代码在新环境和意想不到的环境里都可用
* 典范性 Exemplary: 代码本身鼓励那些为延续这些特点而对它进行的更改

## 创建具有单一职责的类 Single Responsibility Principle, SRP

确保每个类都只有一种单一的、定义明确的职责是实现 TRUE 原则的第一步。

* 易于更改的应用程序由易于重用的类构成。
* 可重用的类是行为严格定义好的可插拔单元，它们不会造成什么纠葛。(就像积木可以随意拼装)
* 拥有多重职责的类难以重用。当我们只想重用它的某些行为时，会发生两种可怕的选择：
  * 该类的多个职责之间紧密耦合，无法重用，所以只能复制其中所需要用到的代码，这会增加额外的维护成本并且会增加错误。
  * 该类的多个职责之间并无紧密耦合，但当我们想要对它更改时，它的每次一次更改都可能破坏所有依赖它的类。

### 确定一个类是否具有单一职责

尝试用一句话来描述类。如果这种描述中出现 “或、和、并” 这样的字，那么这个类就具备了多种职责。

小技巧：可以假设类存在意识，将它的每个方法都改述一个问题，提问应该行之有效。例如：“齿轮先生，请问你的比率是多少呀？” 很合理，而 “齿轮先生，请问你的轮胎尺寸是多少呀？” 就很荒唐。

当类的所有内容都与其中心目标相关联时，就可以说这个类属于高内聚(highly cohesive)，或者具有单一职责。

### 确定何时做出设计决策

* 不必强迫自己过早地做出设计决定。问问自己：如果今天什么都不做，将来的代价会是什么？
* 当什么都不做，而未来的成本与当前的成本相当时，可以推迟决定。
* 新的依赖关系会提供准确的信息，可以基于它做出正确的设计决定。
* 每一个类的结构都是给未来应用程序维护者发出的信息，它揭示了你的设计意图。不管怎样，你今天建立的模式将会被永远地重复。(别指望别人帮你重构)
* 当心，也许你打算将来再重构这部分代码，但其他人现在就需要用到这个类，或者按照这个模式创建新的代码。因为你的同事认为你的意图已经体现在这段代码中。

是 “现在改进” 还是 “将来改进” 这样的矛盾总会存在。好的设计师能够在当前需求和未来可能性之间做出明智的权衡，将成本最小化。

## 编写拥抱变化的代码

### 依赖行为，不依赖数据

每一个细小的行为都只能存在于一个地方。**DRY** 的代码能够容忍变化，因为行为上的任何变化都可以只修改一个地方的代码来实现。

#### 隐藏实例变量

将实例变量包裹在方法里，而不是直接引用变量。

* 好处1：可以将该方法暴漏给其他对象。
* 好处2：可以通过重写方法的实现来行为的更改。
* 好处3：代表数据的变量成了对象。(我无法理解这个好处，太抽象)

```ruby
# BAD
class Gear
  def initialize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def ratio
    @chainring / @cog.to_f
  end
end

# GOOD
class Gear
  attr_reader :chainring, :cog
  def initialize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def ratio
    chainring / cog.to_f
  end
end
```

#### 隐藏数据结构。依赖一个复杂的数据结构是非常糟糕的。


#### 依赖一个复杂的数据结构的缺点

* 发送和接受双方都要了解 data 的数据结构。
* ``diameters`` 方法不仅含有如何计算的公式，还要掌握如何存取钢圈和轮胎的方法。
* 钢圈在索引位置 0 这样的信息不该被复制多次。
* 数据结构一旦更改，每一个引用它的地方都需要更改。

```ruby
class ObscuringReferences
  attr_reader :data
  def initialize(data)
    @data = data
  end

  def diameters
    # 0 代表钢圈，1代表论坛
    data.collect { |cell| cell[0] + (cell[1] * 2) }
  end
end
```

#### 结构和意义分离

* 使用 Ruby 的 Struct 类来包裹结构。
* Struct 是一种将大量属性绑定在一起的便捷方式，它使用了访问器方法，且无需编写类。
* 如果你不得不接受一个混乱的数据结构，至少将这种混乱隐藏起来。
* 重构后 ``diameters`` 方法对数组内部结构一无所知。

```ruby
class ObscuringReferences
  attr_reader :wheels
  def initialize(data)
    @wheels = wheelify(data)
  end

  def diameters
    # 0 代表钢圈，1代表论坛
    wheels.collect { |wheel| wheel.rim + (wheel.tire * 2) }
  end

  private

  Wheel = Struct.new(:rim, :tire)
  def wheelify(data)
    data.collect { |cell| Wheel.new(cell[0], cell[1]) }
  end
end

data = [[622, 20], [622, 23], [599, 30]]
o = ObscuringReferences.new(data)
puts o.diameters
```

### 全面推行单一职责原则

#### 将额外的职责从方法里提取出来，这样可以让它们更容易修改和重用。

```ruby
# 重构前含有两项职责：遍历所有轮子，并计算每一个轮子的直径。注意，这里的描述出现了“并”字。
def diameters
  wheels.collect { |wheel| wheel.rim + (wheel.tire * 2) }
end
```

```ruby
# 重构后
def diameters
  wheels.collect { |wheel| diameter(wheel) }
end

def diameter(wheel)
 wheel.rim + (wheel.tire * 2)
end
```

这样做的好处：

* 暴露出之前隐藏的特性。
* 避免使用注释。如果需要注释，把这些注释的内容提取成一个方法，让方法名来当做注释。
* 鼓励重用。其他程序员会重用这些方法，而不是复制代码。
* 易于移动到另一个类。当出现新功能需要调整代码时，更容易移动。

#### 将类里的额外职责隔离起来

因为不确定目标方向，所以有时候我们不想立刻创建一个新类，因为你的同事可能会在你的基础上去利用那个新类。为了保证当前类的职责单一，并将设计决定推迟到我们有更多需求的时候，我们可以在类的内部嵌入 ``Struct``。

```ruby
class Cear
  attr_reader :chainring, :cog, :wheel
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @wheel = Wheel.new(rim, tire)
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end

  Wheel = Struct.new(:rim, :tire) do
    def diameter
      rim + (tire * 2)
    end
  end
end
```
