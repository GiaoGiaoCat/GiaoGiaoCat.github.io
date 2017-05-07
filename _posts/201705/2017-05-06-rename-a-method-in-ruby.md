---
layout: post
title:  "Rename a method in Ruby: alias_method and remove_method"
date:   2017-05-05 11:30:00
categories: ruby
tags: tip
author: "Victor"
---

一个子类继承自父类，并且重载了父类的一个实例方法，想让子类的实例对象访问父类中的原方法：

假设有如下代码：

```ruby
class Animal
  def speak
    puts "..."
  end
end

class Cat < Animal
  def speak
    puts "Mew..."
  end
end

$ cat = Cat.new
$ cat.speak # Says "Mew..."
```

使用下面的方式，就访问 `Cat` 类继承自 `Animal` 的原始的 `speak` 方法了：

```ruby
class Cat
  alias_method :mew, :speak
  remove_method :speak # removes Cat#speak, not Animal#speak
end

$ cat.speak # Says "..."
$ cat.mew # Says "Mew..."
```


## 原文链接

* [Rename a method in Ruby](https://coderwall.com/p/gutg6g/rename-a-method-in-ruby)
