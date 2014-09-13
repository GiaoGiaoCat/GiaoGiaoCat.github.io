---
layout: post
title:  "Dynamic super-overridable methods in Ruby"
date:   2014-09-04 12:50:00
categories: ruby
tags: metaprogramming
author: "Victor"
---

当你在 library 中重载一个方法 ```super``` 比 ```alias_method``` 更方便:

```ruby
class Post < SomeORM
  attributes :title, :body

  def title
    super || "No title"
  end
end
```

但是，有时候我们需要执行 ```alias_method``` 或 ```alias_method_chain``` 的时候，按照如上的方法就会很不方便:

```ruby
class Post
  attributes :title, :body

  alias_method :old_title, :title

  def title
    old_title || "No title"
  end
end
```

我会建议 library 的作者在可能的情况下尽量使用动态定义方法 ```super-overridable```，这就是本文的内容。

### The problem

通常的实现方法是我们在类中直接使用 ```define_method```

```ruby
class SomeORM
  def self.attributes(*names)
    names.each do |name|
      define_method(name) do
        # Stuff
      end
    end
  end
end
```

library 把方法放入你的 class 里面，例如 ```Post```。所以如果你在 ```Post``` 类中定义一样名称的方法，就会完全替换掉这个旧的方法，并且无法再使用就方法。因此，在元编程的技术中，我们需要使用 ```alias_method```。

### The solution

但是如果 library 可以把方法放入继承链，我们可以定义自己的方法并调用 ```super```。

继承链上都有什么呢？当然是 ```SomeORM``` 的父类以及父类的父类，但是我们不能把方法定义在这些父类中，因为这里定义的方法会让 ```SomeORM``` 的所有子类都继承，而不是只有 ```Post``` 可用。

那么继承链上还有什么呢？答案是 modules

我们有如下的类:

```ruby
class Post < SomeORM
  include SomeModule
end
```

代码的继承链类似于 ```[Post, SomeModule, SomeORM, Object]```。由于在继承链上 ```SomeModule``` 位于自身类和父类之间，所以这里调用 ```self``` 会先查看这些模块，然后再去父类查找。

所以我们可以定义一个模块，并把方法定义在其中:

```ruby
class SomeORM
  def self.attributes(*names)
    mod = Module.new
    include mod

    names.each do |name|
      mod.module_eval do
        define_method(name) do
          # Stuff
        end
      end
    end
  end
end
```

### Polishing the inheritance chain

现在继承链看起来是这样 ```[Post, #<Module:0x007fa0fea7fcf0>, SomeORM, Object]```。

我们创建了一个匿名模块，虽然它可以正常工作，但是将来在调试中可能造成混淆。

并且这种实现方式会为每个 ```attributes``` 定义一个单独的匿名模块。如果你有十个属性，就会在继承链上定义十个匿名方法。虽然它能工作，但是造成了继承链的混乱。

下面这个技巧很好地解决了我们的难题：

```ruby
class SomeORM
  MODULE_NAME = :DynamicAttributes

  def self.attributes(*names)
    if const_defined?(MODULE_NAME, _search_ancestors = false)
      mod = const_get(MODULE_NAME)
    else
      mod = const_set(MODULE_NAME, Module.new)
      include mod
    end

    names.each do |name|
      mod.module_eval do
        define_method(name) do
          # Stuff
        end
      end
    end
  end
end
```

这段代码做了一些有趣的事情，它命名了一个常量 ```Post::DynamicAttributes```，所以现在继承链看起来是 ```Post, Post::DynamicAttributes, SomeORM, Object]```。

我们的代码会检查 ```Post::DynamicAttributes``` 是否存在，如果已经存在那么就重用它。现在，即便 ```Post``` 定义了十个属性，我们也只会在继承链中定义一个模块。

另外 ```const_defined?(MODULE_NAME, _search_ancestors = false)``` 也是很重要的，让我们来看看子类继承 ```Post``` 的情况:

```ruby
class VideoPost < Post
  attribute :timestamp
end
```

因为我们给 ```const_defined?``` 方法的第2个参数传递的值是 ```false```，因此我们不会重用父类的 ```Post::DynamicAttributes``` 模块而是重新定义了一个 ```VideoPost::DynamicAttributes``` 模块来确保我们的属性继承正确。

你也可以不用 ```_search_ancestors``` 的局部变量，直接写成 ```const_defined?(MODULE_NAME, false)```，但是如果 Ruby 版本小于 2.0 的话会有问题。

### Examples in the wild

这里有两个使用这种技巧的例子 [Traco](https://github.com/barsoom/traco/blob/master/lib/traco/translates.rb) 和 [Minimapper](https://github.com/joakimk/minimapper/commit/e3ebe9d3148a26c51d81ddc03a3cab567a65ba46)

### 相关链接

* [原文](http://thepugautomatic.com/2013/07/dsom/)
