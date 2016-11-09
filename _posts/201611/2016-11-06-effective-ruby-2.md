---
layout: post
title:  "Effective Ruby 2 - 类、对象和模块（下）"
date:   2016-11-06 11:30:00
categories: ruby
tags: learningnote
author: "Victor"
---

## 类、对象和模块

### 11 通过在模块中嵌入代码来创建命名空间

* 通过在模块中嵌入代码来创建命名空间
* 让命名空间结构和目录结构相同
* 如果创建命名空间的模块已经定义过，可以在类定义中直接使用模块和类路径分隔符 `class Notebooks::Binding`
* 如果使用时可能出现歧义，可使用 `::` 来限定定级常量，比如 `::Array`

一个典型的应用场景是，现在入口源文件中定义了命名空间模块，随后加载所有剩下的源文件。

下面也是一个可能会遇到的坑。根据词法作用域，`initialize` 方法中的 `Array` 实际上找到的是 `Cluster::Array`。

```ruby
module Cluster
  class Array
    def initialize(n)
      @disks = Array.new(n) { |i| "disk#{i}" }
      # Oops, wrong Array! SystemStackError!
    end
  end
end
```

### 12 理解等价的不同用法

先读一下 [CodeCademy Ruby 学习笔记](/ruby/codecademy-ruby/) 中的附录2，在此基础上进一步介绍这4种方法的区别。

* 绝对不要重载 `equal?` 方法，该方法比较的是两个对象的内存地址
* `==` 类意义上的相等，需要每个类自己定义其实现，可以根据业务需求覆盖此方法，简单理解为比较两个对象的值是否相同
* `===` case 语句使用该方法
* `eql?` 别名到 `==` 方法，需要时可以重载该方法改变 Hash 比较时的表现

### 13 通过 `<=>` 操作符实现比较和比较模块

* 通过定义 `<=>` 操作符和引入 Comparable 模块实现对象的排序
* 当编写一个比较操作符时，通常的做法是将参数命名为 `other`
* 当接受者和要比较的参数不是同一个类的实例时，`<=>` 应该返回 `nil`
* 接受者比参数小，返回 -1；接受者比参数大，返回 1；相等返回 0；
* 当编写 `<=>` 时，通常的做法是将比较方法代理给对象的实例变量

```ruby
class Version
  include Comparable

  attr_reader :major, :minor, :patch

  def initialize(version)
    @major, @minor, @patch = version.split('.').map(&:to_i)
  end

  def <=>(other)
    return nil unless other.is_a(Version)

    [ major <=> other.major,
      minor <=> other.minor,
      patch <=> other.patch
    ].detect { |n| n.zero? } || 0
  end
end
```

```
vs = %w(1.0.0 1.11.1 1.9.0).map { |v| Version.new(v) }
vs.sort

a = Version.new('2.1.1')
b = Version.new('2.10.3')
a > b #=> true
Version.new('2.8.0').between?(a, b) #=> true
```

### 14 通过 `protected` 方法共享私有状态

* 一个对象的 `protected` 方法若要被显示接受者调用，除非该对象与接受者是同类对象或具有相同的定义该 `protected` 方法的超类

### 15 优先使用实例变量而非类变量

类变量通常的用途是实现单例模式（不要和上文的 Ruby 单例类混淆）。比如配置类，你希望它只存在一个实例，所有的代码访问这个唯一的实例。单例类不能存在一个以上的实例，所以典型的做法是使 `new`, `dup`, `clone` 方法私有化。

```ruby
class Singleton
  private_class_method(:new, :dup, :clone)

  def self.instance
    @@single ||= new
  end
end
```

访问这个对象的唯一方法是通过 `instance` 类方法。类变量可以被所有子类共享，意思是这些子类的任何实例都可以访问并修改这些共享的类变量。而类的实例变量只能被类的实例所访问和修改，因此出现了以下的区别。

```ruby
class Singleton
  private_class_method(:new, :dup, :clone)

  def self.instance
    @@single ||= new
  end
end

class Configuration < Singleton
end

class Database < Singleton
end

Configuration.instance #=> #<Configuration>
Database.instance #=> #<Configuration> 因为 @@single 已经在之前被赋值了
```

```ruby
class Singleton
  private_class_method(:new, :dup, :clone)

  def self.instance
    @single ||= new
  end
end

class Configuration < Singleton
end

class Database < Singleton
end

Configuration.instance #=> #<Configuration> 这里 @signle 是 Configuration 的类对象的实例变量
Database.instance #=> #<Database> 这里 @signle 是 Database 的类对象的实例变量，和上面互补影响
```

改用类的实例变量之后，不但打破了类和子类的共享关系，还提供了更多的封装性。你可以根据需要提供 `@single` 的访问方法。

* 优先使用实例变量而非类变量
* 类也是对象，所以她们拥有自己的私有实例变量集合
