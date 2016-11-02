---
layout: post
title:  "Effective Ruby 2 - 类、对象和模块"
date:   2016-11-01 02:30:00
categories: ruby
tags: learningnote
author: "Victor"
---

## 类、对象和模块
### 06 了解 Ruby 如何构建继承体系

* 对象是变量的容器。这些变量被称为实例变量并代表了一个对象的状态。
* 类是方法和常量的容器。这些方法被称为实例方法并代表了类的所有实例对象的行为。
* 超类 `superclass` 就是父类的别名。
* 类是 `Class` 的实例，模块是 `Module` 的实例。模块没有 `new` 方法。
* 单例类 `singleton class` 是继承体系中的一个匿名且不可见的类。
* 接受者是调用方法的那个对象。

#### 单例类
单例类在 Ruby 中承担了重要的角色，它们是 Ruby 自身根据需要动态创建的。你不能创建一个单例类的实例。唯一需要记住的是：**单例类是没有名字、被加以限制的常规类。**

当使用 `include` 方法将模块引入类时，Ruby 会创建一个单例类并将它插入类的继承体系中。这个匿名不可见的单例类链向该模块，因此它们共享了实例方法和常量。

因为单例类是匿名不可见的，因此 `superclass` 和 `class` 方法都会跳过它。

当模块被包含进类时，以后进先出（LIFO）的方式。Ruby 查找一个方法时，它以逆序访问每个模块，最后包含的模块最先访问到。如果不使用 `perpend` 方法的话，正常情况下 Ruby 的继承体系决定了 **模块永远不会重载类中的方法。**

#### 单例方法
单例类仅仅服务一个对象，它的方法只能在这个对象上使用，在其它任何对象上都不能调用。

```ruby
customer = Customer.new

def customer.name
  'Leonard'
end
```

Ruby 执行这段代码时，它创建一个单例类，将 `name` 方法作为其实例方法，随后将这个匿名类作为 `customer` 对象的类进行插入。虽然现在 `customer` 对象的类是单例类，Ruby 的内省方法 `class` 方法仍将跳过它并返回 `Customer`。这样 Ruby 的实现变得很简单，查找 `name` 方法时，仅需要根据继承体系遍历一下就行。

所以，类方法也是作为单例类的实例方法存储的。

```ruby
class Customer < Person
  def self.where_am_i?
    #...
  end
end

#=> Customer.singleton_class.instance_methods(false)
#=> [:where_am_i?]
#=> Customer.singleton_class.superclass
#=> #<Class:Person>
```

#### 查找方法的过程
1. 找到接受者的类，它可能是一个隐藏的单例类。
2. 查找该类中储存的实例方法列表。找到就停止。
3. 顺着继承体系向上找到超类并重复步骤2，这个超类可能是单例类，但通过 `superclass` 方法这个单例类不可见。
4. 重复步骤2和3，直到体系的顶点。
5. 返回步骤1，找 `method_missing` 方法。

* `singleton_class` 返回接受者的单例类，如果不存在就创建它。
* `ancestors` 返回组成了继承体系的所有类和模块的数组，只能在类和模块上调用，会跳过单例类。
* `included_modules` 返回和 `ancestors` 方法一样的数组，不过其中所有的类都被过滤掉了。

#### 总结
* Ruby 要寻找一个方法，只需要向上搜索这个类体系。如果没有找到，就从起点开始搜索 `method_missing` 方法。
* 包含模块时 Ruby 会创建单例类，并将其插入在继承体系中包含它的类的上方。
* 单例方法（类方法和针对对象的方法）储存于单例类中，它也会被插入继承体系中。

### 07 了解 `super` 的不同行为
* 当你想重载继承体系中的一个方法时，`super` 可以帮你调用它，`super` 后面跟随的参数会传递给重载方法。
* 不加括号的无参调用 `super` 等价于将宿主方法的所有参数传递给重载方法，如果存在代码块也一并传递。
* 如果希望使用 `super` 并且不向重载方法传递任何参数，必须使用空括号，即 `super()`。

```ruby
class SuperSilliness < SillyBase
  def m1(x, y)
    super(1, 2) # Call with 1, 2.
    super(x, y) # Call wit x, y.
    super x ,y # Same as above.
    super # Same as above.
    super() # Call without arguments.
  end
end
```

需要注意的是 `super` 能对模块中的方法进行重载，如果你希望重载的是超类中的方法，而实际上重载的是模块中的方法，说明设计上存在问题，不应该使用继承而改用组合。

另外一个问题是 `super` 和 `method_missing` 的相互作用。

```ruby
class SillyBase
  def method_missing
    # ...
  end
end

class SuperSilliness < SillyBase
  def m1(x, y)
    super
  end
end
```

在 `SuperSilliness` 中调用的 `super` 并没在继承体系中的超类中找到可重载的方法，但是因为超类中定义了 `method_missing`，当 `super` 调用失败时，自定义的 `method_missing` 方法将丢弃一些有用的信息，在后面的 30 条建议中有替代解决方案。
