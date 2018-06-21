---
layout: post
title:  "Enumerable & Enumerator in Ruby"
date:   2014-11-19 10:00:00
categories: ruby
tags: enumerable
author: "Victor"
---

## Enumerable

在 Ruby 中如果你想编写一次代码和一堆不相关的类之间共享它，你可以把它放在模块在 "mixed in" 进入你的类。你可以通过 ```Object#included_modules``` 来查看都混入了什么模块。

```ruby
Array.included_modules #=> [Enumerable, PP::ObjectMixin, Kernel]
String.included_modules #=> [Comparable, PP::ObjectMixin, Kernel]
```

如果想在你的类中使用 **Enumerable**，你必须定义一个 **#each** 方法，这个方法需要在代码块中 **yield** 每一个元素。

```ruby
class Colors
  include Enumerable

  def each
    yield "red"
    yield "green"
    yield "blue"
  end
end

c = Colors.new
c.map { |i| i.reverse } #=> ["der", "neerg", "eulb"]
```

可以使用 **#instance_methods** 方法，来查看 **Enumerable** 提供的实例方法：

```ruby
Enumerable.instance_methods.sort #=>
  [ :all?, :any?, :chunk, :collect, :collect_concat, :count, :cycle,
    :detect, :drop, :drop_while, :each_cons, :each_entry, :each_slice,
    :each_with_index, :each_with_object, :entries, :find, :find_all,
    :find_index, :first, :flat_map, :grep, :group_by, :include?, :inject,
    :map, :max, :max_by, :member?, :min, :min_by, :minmax, :minmax_by,
    :none?, :one?, :partition, :reduce, :reject, :reverse_each, :select,
    :slice_before, :sort, :sort_by, :take, :take_while, :to_a, :zip ]
```

## Enumerator

Enumerable 模块定义了很多方便的方法，但是作为模块中其他方法的基础，将遍历元素的方法限定为 each 方法，这一点有些不太灵活。

String 对象有 `each_byte`, `each_line`, `each_char` 等用于循环的方法，如果这些方法都能使用 `each_with_index`, `collect` 等 Enumerable 模块的方法就方便多了。而 Enumerator 类就是解决这个问题的。

Enumerator 类能以 `each` 方法以外的方法为基础，执行 Enumerable 模块定义的方法。使用 Enumerator 类后，我们就可以用 `String#each_line` 方法替代 `each` 方法，从而执行 Enumerable 模块的方法。

另外，在不带块的情况下，大部分 Ruby 原生的迭代器在调用时都会返回 Enumerator 对象，因此可以在 `each_line`, `each_byte` 等方法的返回结果继续使用 `map` 等方法。

```ruby
str = "AA\nBB\nCC\n"
p str.each_line.class #=> Enumerator
p str.each_line.map { |line| line.chop } #=> ["AA", "BB", "CC"]
```

### 相关阅读

* [A Guide to Ruby Collections III: Enumerable and Enumerator](http://www.sitepoint.com/guide-ruby-collections-iii-enumerable-enumerator/)
* [Building Enumerable & Enumerator in Ruby](https://practicingruby.com/articles/building-enumerable-and-enumerator)
* [Ruby Enumerator类](http://www.tuicool.com/articles/NJzIZrz)
* [ruby之enumerator](http://fansofjava.iteye.com/blog/718382)
* [学习ruby之Enumerable模块](http://it.chinawin.net/softwaredev/article-11947.html)
* [Ruby中Enumerable#inject用法示范](http://puffsun.iteye.com/blog/1986421)
* [Ruby: inject](http://blog.jayfields.com/2008/03/ruby-inject.html)
* [Enumerable](http://www.kuqin.com/rubycndocument/man/built-in-class/module_enumerable.html)
* [Ruby 2.0 Enumerable::Lazy](http://railsware.com/blog/2012/03/13/ruby-2-0-enumerablelazy/)
