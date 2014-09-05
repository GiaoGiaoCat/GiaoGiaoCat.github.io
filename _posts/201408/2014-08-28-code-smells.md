---
layout: post
title:  "Code Smells"
date:   2014-08-28 14:30:00
categories: ruby
tags: refactoring
author: "Victor"
---

## 1. Duplicate code 重复的代码

如果你在一个以上的地点看到相同的程序结构，那么当可肯定：设法将它们合而为一，程序会变得更好。

* 同一个 class 内的两个函数中含有重复的代码段
* 两个兄弟 class 的函数中含有重复的代码段
* 两个毫不相关的 class 内出现重复的代码段

**注意：重复的代码是多数潜在BUG的温床！**

### 用到的重构方法简介:

* Extract Method __提炼函数__: 将重复的代码放到一个函数中, 并让函数名称清晰的解释函数的用途。
* Pull Up Method __函数上移__: 将函数从子类移动到父类中。
* From Template Method __塑造模板函数__: 不同子类中某些函数执行相似操作,细节上不同, 可以将这些操作放入独立函数中, 这些函数名相同, 将函数上移父类中。
* Substitute Algorithm __替换算法__: 将函数的本体替换成另外一个算法。
* Extract Class __提炼类__: 建立一个新类, 将相关的函数和字段从旧类搬移到新类。

### 解决方案:

#### 同类函数重复代码

* 使用 Extract Method __提炼函数__ 方法提炼出重复的代码, 两个函数同时调用这个方法, 代替使用相同的表达式。

#### 兄弟子类重复代码

* 代码相同解决方案: 对两个子类 使用 Extract Method __提炼函数__ 方法, 然后将提炼出来的代码 使用 Pull Up Method __函数上移__ 方法, 将这段代码定义到父类中去;
* 代码相似解决方案: 使用 Extract Method __提炼函数__ 方法 将相似的部分 与 差异部分 分割开来, 将相似的部分单独放在一个函数中;
* 进一步操作: 进行完上面的操作之后, 可以运用 From Template Method __塑造模板函数__ 获得一个 Template Method 设计模式, 使用模板函数将相似的部分设置到模板中, 不同的部分用于模板的参数等变量;
* 算法切换: 如果模板中函数的算法有差异, 可以选择比较清晰的一个, 使用 Substitute Algorithm __替换算法__ 将不清晰的算法替换掉;

#### 不相干类出现重复代码

* 使用Extract Class __提炼类__ 方法, 将重复的代码提炼到一个重复类中去, 然后在两个类中 使用这个提炼后的新类;
* 提炼类存在方式: 将提炼后的代码放到两个类中的一个, 另一个调用这个类, 如果放到第三个类, 两个类需要同时引用这个类;

## 2. Long method 过长的方法

拥有短函数的对象会活的比较好、比较长。

* 程序愈长就愈难理解
* 函数过长阅读起来也不方便
* 小函数的价值：解释能力、共享能力、选择能力，并且可维护性更高

### 用到的重构方法简介:

* Extract Method __提炼函数__: 将代码放到一个新函数中，函数名清晰的说明函数的作用。
* Replace Temp with Query __以查询取代临时变量__: 程序中将表达式结果放到临时变量中，可以将这个表达式提炼到一个独立函数中，调用这个新函数去替换这个临时变量表达式, 这个新函数就可以被其它函数调用;
* Introduce Parameter Object __引入参数对象__: 将参数封装到一个对象中，以一个对象取代这些参数。
* Preserve Whole Object __保持对象完整__: 从某个对象中取出若干值, 将其作为某次函数调用时的参数，由原来的传递参数改为传递整个对象，类似于 Hibernate。
* Replace Method with Method Object __以函数对象取代函数__: 大型函数中有很多参数和临时变量，将函数放到一个单独对象中，局部变量和参数就变成了对象内的字段, 然后可以在 同一个对象中 将这个 大型函数 分解为许多小函数。
* Decompose Conditional __分解条件表达式__: 将 ```if then else while``` 等语句的条件表达式提炼出来，放到独立的函数中去。

### 何时重构:

1. 当我们需要添加注释的时候，就应该将要注释的代码写入到一个独立的函数中，并以代码的用途命名。
2. 尽可能分解函，即使函数中只有一行代码，哪怕函数调用比函数还要长，只要函数名能解释代码用途就可以。
3. 函数长度不是关键，关键在于函数是做什么，和如何做。

**为什么命名很重要：看代码的时候经常转换上下文查看，为函数起一个容易懂的好名称, 一看函数名就能明白函数的作用。**

### 解决方案:

#### 常用分解方法

* Extract Method __提炼函数__ 适用于 99% 的过长函数情况，只要将函数中冗长的部分提取出来，放到另外一个函数中即可。

#### 参数过多情况

如果函数内有大量的参数和临时变量，就会对函数提炼形成阻碍，这时候使用 Extract Method __提炼函数__ 方法就会将许多参数和临时变量当做参数传入到提炼出来的函数中。

* 消除临时变量: 使用 Replace Temp with Query __以查询取代临时变量__ 方法消除临时元素。
* 消除过长参数: 使用 Introduce Parameter Object __引入参数对象__ 和 Preserve Whole Object __保持对象完整__ 方法可以将过长的参数列变得简洁一些。
* 杀手锏: 如果使用了上面消除临时变量和过长参数的方法之后，还存在很多参数和临时变量，此时就可以使用 Replace Method with Method Object __以函数对象取代函数__。

#### 提炼代码技巧

* 寻找注释: 注释能很好的指出 __代码用途__ 和 __实现手法__ 之间的语义距离，代码前面有注释，就说明这段代码可以替换成一个函数，在注释的基础上为函数命名，即使注释下面只有一行代码，也要将其提炼到函数中。
* 条件表达式: 当 ```if else``` 语句, 或者 ```while``` 语句的条件表达式过长的时候，可以使用 Decompose Conditional __分解条件表达式__ 方法，处理条件。
* 循环代码提炼: 当遇到循环的时候，应该将循环的代码提炼到一个函数中去。

## 3. Feature envy 依恋情节

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

## 5. Comments 过多的注释

过多注释的代码段，往往都是因为那段代码比较糟糕，散发着一股恶臭。

**原则：当你感觉需要写注释时，请尝试重构，试着让所有注释都变得多余。**

## 6. Divergent Change 发散式变化

我们希望软件能够更容易被修改。一旦需要修改，我们希望能够跳到系统的某一点，只在该处做修改。如果不能做到这点，而要同时修改一个类中的多个方法，就意味着程序出现了 __发散式变化__。

发散式变化：一个类受多种变化的影响

* 数据库新加一个字段，同时修改三个函数：Load、Insert、Update
* 新加一个角色二进制，同时修改四处
* ...

**原则：针对某一外界变化的所有相应修改，都只应该发生在单一类中**

### 用到的重构方法:

* Extract Class __提炼类__

### 解决方案:

* 找出造成发散变化的原因, 使用 Extract Class __提炼类__ 将需要修改的方法集中到一个类中。


## 7. Primitive Obsession 基本型别偏执

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

## 8. Large Class 过大类

如果想利用单一类做太多事情，其内往往就会出现太多的成员变量。

* 提取完成同一任务的相关变量到一个新的类
* 干太多事情的类，可以考虑把责任委托给其他类

**注意：一个类如果拥有太多的代码，也是代码重复、混乱、死亡的绝佳滋生地点。**

### 用到的重构方法简介:

* Extract Class __提炼类__: 建立一个新类，将相关的函数和字段从旧类搬移到新类。
* Extract Subclass __提炼子类__: 一个类中的某些特性只能被一部分实例使用到，可以新建一个子类，将只能由一部分实例用到的特性转移到子类中。
* Extract Interface __提炼接口__: 多个客户端使用类中的同一组代码，或者两个类的接口有相同的部分，此时可以将相同的子集提炼到一个独立接口中。
* Duplicate Observed Data __复制被监视的数据__: 一些领域数据放在GUI控件中，领域函数需要访问这些数据。将这些数据复制到一个领域对象中，建立一个观察者模式，用来同步领域对象 和 GUI对象的重要数据。

### 解决方案:

#### 实例变量太多解决方案

如果一个类的职能太多，在单个类中做太多的事情，这个类中会出现大量的实例变量。而往往 Duplicate Code __重复代码__ 与 Large Class __过大的类__ 是一起产生的。

* 选择相关变量: 选择类中相关的变量提炼到一个新类中，一般前缀，后缀相同的变量相关性较强，可以将这些相关性较强的变量提炼到一个类中。
* 子类提炼: 如果一些变量适合作为子类，使用 Extract Subclass __提炼子类__ 方法，可以创建一个子类继承该类，将提炼出来的相关变量放到子类中。
* 多次提炼: 一个类中定义了20个实例变量，在同一个时刻，只使用一部分实例变量，比如在一个时刻只使用5个，在另一时刻只使用4个。我们可以将这些实例变量多次使用 Extract Class __提炼类__ 和 Extract Subclass __提炼子类__ 方法。

#### 代码太多解决方案

太多的代码是 Duplicate Code __重复代码__, 混乱, 项目崩溃的源头。

* 简单解决方案: 使用 Extract Method __提炼函数__ 方法, 将重复代码提炼出来。
* 提炼类代码技巧: 使用 Extract Class __提炼类__ 和 Extract Subclass __提炼子类__ 方法对类的代码进行提炼, 先确定客户端如何使用这个类, 之后运用 Extract Interface __提炼接口__ 为每种使用方式提炼出一个接口, 可以更清楚的分解这个类。
* GUI类提炼技巧: 使用 Duplicate Observed Data __复制被监视的数据__ 方法, 将数据和行为提炼到一个独立的对象中, 两边各保留一些重复数据, 用来保持同步。


## 9. Long Parameters List 过长的参数列表

太长的参数列表难以理解，太多参数会造成前后不一致、不易使用，而且你需要更多数据时，就要修改函数参数结构。如果使用对象传递函数，如果需要更多的参数，只需要在对象中添加字段即可。

**原则：参数不超过3个！**

### 使用到的重构方法简介:

* Replace Parameter with Method __以函数取代参数__: 对象调用 函数1，将结果作为 函数2 的参数，函数2 内部就可以调用 函数1，不用再传递参数了。
* Preserve Whole Object __保持对象完整__: 将对象中的一些字段是函数的参数，直接将对象作为函数的参数，由传递多个参数改为传递封装好的对象。
* Introduce Parameter Object __引入参数对象__: 将函数参数封装在一个对象中。

# 10. Shotgun Surgery 霰弹式修改

如果每遇到某种变化，你都必须在许多不同的类内做出许多小修改以响应之。如果需要修改的代码散布四处，你不但难以找到它们，也很容易忘记某个重要的修改。

霰弹式修改：一种变化引起多个类相应的修改

注意 __霰弹式修改__ 与 __发散式变化__ 区别: 发散式变化是在一个类受多种变化影响, 每种变化修改的方法不同, 霰弹式修改是一种变化引发修改多个类中的代码。

### 使用到的重构方法简介:

* Move Method __搬移函数__: 类A中的方法A与类B交流频繁，在类B中创建一个与方法A相似的方法B，从方法A中调用方法B，或者直接将方法A删除。
* Move Field __搬移字段__: 类A中的字段A经常被类B用到，在类B中新建一个字段B，在类B中尽量使用字段B。
* Inline Class __内联化类__: 类A没有太多功能，将类A的所有特性搬移到类B中，删除类A。

### 解决方案:

* 代码集中到某个类中: 使用 Move Method __搬移函数__ 和 Move Field __搬移字段__ 把所有需要修改的代码放进同一个类中。
* 代码集中到新创建类中: 没有合适类存放代码, 创建一个类, 使用 Inline Class __内联化类__ 方法将一系列的行为放在同一个类中。
* 造成分散式变化: 上面的两种操作会造成 Divergent Change __发散式变化__, 使用 Extract Class __提炼类__ 处理发散式变化。

## 11. Switch Statement Switch 语句

面向对象程序的一个最明显特征就是：少用 switch 语句。从本质上说 switch 语句的问题在于重复。

**原则：看到 switch 你就应该考虑使用多态来替换它。**

## 12. Temporary Field 暂时成员变量

有时你会看到这样的对象：其内某个成员变量仅为某种特定的情形而设。这样的代码容易让人不解，因为你通常认为对象在所有时候都需要它的所有变量。

在变量未被使用的情况下猜测当初设置目的，会让你发疯。

一个对象的属性可能只在某些情况下才有意义。这样的代码将难以理解。专门建立一个对象来持有这样的孤儿属性，把只和他相关的行为移到该类。最常见的是一个特定的算法需要某些只有该算法才有用的变量。

## 最后

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
* [21种代码的坏味道](http://www.knowsky.com/362766.html)
* [Bad Smell(代码的坏味道)](http://blog.csdn.net/sulliy/article/details/6635596)
