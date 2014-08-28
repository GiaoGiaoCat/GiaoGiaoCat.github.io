---
layout: post
title:  "Code Smells"
date:   2014-08-28 14:30:00
categories: ruby
tags: refactoring editor
author: "Victor"
---

### 1. Duplicate code 重复的代码

如果你在一个以上的地点看到相同的程序结构，那么当可肯定：设法将它们合而为一，程序会变得更好。

* 同一个 class 内的两个函数中含有重复的代码段
* 两个兄弟 class 的成员函数中含有重复的代码段
* 两个毫不相关的 class 内出现重复的代码段

**注意：重复的代码是多数潜在BUG的温床！**

### 2. Long method 过长的方法

拥有短函数的对象会活的比较好、比较长。

* 程序愈长就愈难理解
* 函数过长阅读起来也不方便
* 小函数的价值：解释能力、共享能力、选择能力

**原则：每当感觉需要以注释来说明点什么的时候，我们就把需要说明的东西写进一个独立的函数中。记着，起个好名字！**

### 3. Feature envy 依恋情节

函数对某个类的兴趣高过对自己所处类的兴趣，就会产生对这个类的依恋情节，造成紧耦合。

**原则：判断哪个类拥有最多被此函数使用的数据，然后将这个函数和那些数据摆在一起。**

**原则：将总是变化的东西放在一块。**

```ruby
# BAD
class A
  attr_reader :x, :y, :z

  def initialize x, y, z
    @x = x
    @y = y
    @z = z
  end

  def some_function
    y / z
  end
end

class B
  attr_reader :a

  def initialize a
    @a = a
  end

  def my_method
    a.x + a.y * a.z ** a.some_function
  end
end

# GOOD
class A
  attr_reader :x, :y, :z

  def initialize x, y, z
    @x = x
    @y = y
    @z = z
  end

  def some_function
    y / z
  end

  def special_method
    x + y * z ** some_function
  end
end

class B
  attr_reader :a

  def initialize a
    @a = a
  end

  def my_method
    a.special_method
  end
end
```

### 4. Data clumps 数据泥团

有些数据项，喜欢成群结队地待在一块。那就把它们绑起来放在一个新的类里面。这样就可以：

* 缩短参数列表
* 简化函数调用

```ruby
class Point
  attr_reader :x, :y, :z

  def initialize x, y, z
    @x = x
    @y = y
    @z = z
  end

  #BAD
  def some_function _x, _y, _z
    # Do something with it
  end

  #GOOD
  def some_function point
    # Do something with it
  end
end
```

### 5. Comments 过多的注释

过多注释的代码段，往往都是因为那段代码比较糟糕，散发着一股恶臭。

**原则：当你感觉需要写注释时，请尝试重构，试着让所有注释都变得多余。**

### 6. Divergent Change 发散式变化

我们希望软件能够更容易被修改。一旦需要修改，我们希望能够跳到系统的某一点，只在该处做修改。如果不能做到这点，你就嗅出“坏味道：发散式变化”或“坏味道：霰弹式修改”。

发散式变化：一个类受多种变化的影响

* 数据库新加一个字段，同时修改三个函数：Load、Insert、Update
* 新加一个角色二进制，同时修改四处
* ...

**原则：针对某一外界变化的所有相应修改，都只应该发生在单一类中**

### 7. Primitive Obsession 基本型别偏执

代码中有很多基本数据类型的数据。

**原则：如果看到一些基本类型数据，尝试定义一种新的数据类型，符合它当前所代表的对象类型。**

```ruby
class Product
  attr_reader :name, :color, :price
  def initialize name, color, price
    @name = name
    @color = color
    @price = price
  end
end

# BAD
[ "Shirt", "Pink", 10 ]

# GOOD
Product.new "Shirt", "Pink", 10
```

### 8. Large Class 过大类

如果想利用单一类做太多事情，其内往往就会出现太多的成员变量。

* 提取完成同一任务的相关变量到一个新的类
* 干太多事情的类，可以考虑把责任委托给其他类

**注意：一个类如果拥有太多的代码，也是代码重复、混乱、死亡的绝佳滋生地点。**

### 9. Long Parameters List 过长的参数列表

太长的参数列表难以理解，太多参数会造成前后不一致、不易使用，而且你需要更多数据时，就不得不修改它。

**原则：参数不超过3个！**

### 10. Shotgun Surgery 霰弹式修改

如果每遇到某种变化，你都必须在许多不同的类内做出许多小修改以响应之。如果需要修改的代码散布四处，你不但难以找到它们，也很容易忘记某个重要的修改。

霰弹式修改：一种变化引起多个类相应的修改

### 11. Switch Statement Switch 语句

面向对象程序的一个最明显特征就是：少用 switch 语句。从本质上说 switch 语句的问题在于重复。

**原则：看到 switch 你就应该考虑使用多态来替换它。**

### 12. Temporary Field 暂时成员变量

有时你会看到这样的对象：其内某个成员变量仅为某种特定的情形而设。这样的代码容易让人不解，因为你通常认为对象在所有时候都需要它的所有变量。

在变量未被使用的情况下猜测当初设置目的，会让你发疯。

一个对象的属性可能只在某些情况下才有意义。这样的代码将难以理解。专门建立一个对象来持有这样的孤儿属性，把只和他相关的行为移到该类。最常见的是一个特定的算法需要某些只有该算法才有用的变量。

### 最后

尚有如下情况未列入，欢迎大家补充。

* Parallel Inheritance Hierarchies
* Speculative Generality
* Message Chain
* Middle Man
* Inappropriate Intimacy
* Alternative Classes with Different Interfaces
* Incomplete Library Class
* Data Class

### 相关阅读和参考

* [MountainWest RubyConf 2013 Code Smells: Your Refactoring Cheat Codes by John Pignata](http://www.youtube.com/watch?v=q_qdWuCAkd8)
* [http://rubylearning.com/blog/2011/03/01/how-do-i-smell-ruby-code/](http://rubylearning.com/blog/2011/03/01/how-do-i-smell-ruby-code/)
* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [代码的坏味道](http://www.cnblogs.com/mywolrd/archive/2012/04/24/2467395.html)
