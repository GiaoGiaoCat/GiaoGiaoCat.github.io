---
layout: post
title:  "5个很少用到的 Enumerators 方法"
date:   2015-04-10 11:00:00
categories: ruby
tags: enumerable
author: "Victor"
---

[Enumerable](http://ruby-doc.org/core-2.1.5/Enumerable.html#method-i-entries) 有很多非常有用的方法，希望你在打算自己实现它们之前先看看这个模板都提供了哪些。

## Let's partition

`#partition` 可以把一个集合分割成两部分。下面是一个例子：把一个数组中的人和动物分割成两个数组。

```ruby
class Animal
  def is_animal?
    true
  end
end
class Human
  def is_animal?
    false
  end
end

[Animal.new, Animal.new, Human.new, Human.new, Animal.new].partition { |c| c.is_animal? }

#=> [[#<Animal:0x007fb019830d08>, #<Animal:0x007fb019830ce0>, #<Animal:0x007fb019830bf0>], [#<Human:0x007fb019830c68>, #<Human:0x007fb019830c18>]]
```

## Each with object you say?

在迭代一个数组的时候，我们需要给代码块传递进一个对象。

下面的例子中我们有一个 `Person` 实例，并且它有 `greet` 方法。我们期望在迭代的时候给代码块传入一个字符串。

```ruby
class Person
  def initialize(name)
    @name = name
  end
  def greet(greeter)
    puts "#{greeter} #{@name}"
  end
end

[Person.new("Peter"), Person.new("Meg"), Person.new("Louis")].each_with_object("Hello") { |i, a| i.greet(a) }

#=>
# Hello Peter
# Hello Meg
# Hello Louis
```

## Finding maximal price

我们有一组商品列表，我们想返回其中价格最高的。

```ruby
class Product
  def initialize(price)
    @price = price
  end
  attr_reader :price
end

[Product.new(100), Product.new(120), Product.new(1000)].max_by(&:price)
=> #<Product:0x007fb019861228 @price=1000>
```

## What about lowest and highest price?

同样，如果想要找到最高和最低价格也很容易。

```ruby
class Product
  def initialize(price)
    @price = price
  end
  attr_reader :price
end

[Product.new(100), Product.new(120), Product.new(1000)].minmax_by(&:price)
=> [#<Product:0x007fb01887a418 @price=100>, #<Product:0x007fb01887a3c8 @price=1000>]
```

## Take while we can!

现在让我们玩一个警察和劫匪的游戏。我们打算抢劫对面的一堆人。目标是当遇到警察的时候，就停止打劫并跑路。

注意下面例子返回的结果，体会一下 `take_while` 和 `select` 的区别。

```ruby
class Person
  def can_be_robbed?
    true
  end
end

class Cop
  def can_be_robbed?
    false
  end
end

[Person.new, Person.new, Cop.new, Person.new].take_while(&:can_be_robbed?)

=> [#<Person:0x007fa21a848848>, #<Person:0x007fa21a848820>]
```

## 相关阅读

* [5 less used Enumerators of Ruby](http://codingwithaxe.com/5-less-used-enumerators-of-ruby)
* [Ruby: How to Iterate "the Right Way"](http://jeromedalbert.com/ruby-how-to-iterate-the-right-way/)
