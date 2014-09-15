---
layout: post
title:  "Ruby 中的委托"
date:   2014-09-15 11:30:00
categories: ruby
tags: tip
author: "Victor"
---

## 委托的基本概念

### 什么是委托

委托是一种特殊的类型（class），用途是来实现对一种方法的封装。在某种事件发生时，自动调用该方法。用最通俗易懂的话来讲，你就可以把委托看成是用来执行方法（函数）的一个东西。

### 如何使用委托

在使用委托的时候，你可以像对待一个类一样对待它。即先声明，再实例化。只是有点不同，类在实例化之后叫对象或实例，但委托在实例化后仍叫委托。


### 委托的好处

好处显然易见，它使用户可以自定义自己的方法实现，通过封装，在相应事件激发时调用你定义的方法，实现你的功能。


## DelegateClass

负责定义并返回一个委托类，该类会委托 superclass 类的某个对象来执行实例方法。

当不需要动态修改委派接受对象的时候使用 ```DelegateClass```。

```ruby
require 'delegate'

class Assistant
  def initialize(name)
    @name = name
  end

  def read_email
    "(#{@name}) It is mostly spam."
  end
end

class Manager < DelegateClass(Assistant)
  def initialze(assistant)
     super(assistant)
  end

  def attend_meeting
    puts "please hold my call"
  end
end
```

```ruby
assistant= Assistant.new("Bill Gate")
mgr = Manager.new(assistant)
mgr.read_email #(Bill Gate) It is mostly spam.
mgr.attend_meeting # please hold my call
```


## SimpleDelegator

### 方法介绍

当一个对象在自己的生命周期中，需要把动作委托给不同的对象时，使用 ```SimpleDelegator```。

类方法：

```ruby
SimpleDelegator.new(obj) # 生成一个对象, 它委托obj来执行自身所拥有的方法。
```

实例方法：

```ruby
__getobj__ # 返回接受委托的对象。
__setobj__ # 将接受委托的对象变为obj。
```

请注意，因为只有在生成时才会进行委托方法的定义，所以即使接受委托的对象和obj之间存在实例方法上的差异，也无法再次设定委托实例方法。因此，这里的建议是新的委托对象最好和原始的委托对象保持类型一致。

### 例子

```ruby
# example 1:
names = SimpleDelegator.new(%w{James Edward Gray II})
puts names[1] # => Edward
names.__setobj__(%w{Gavin Sinclair})
puts names[1] # => Sinclair
```

```ruby
# example 2:
class SuperArray < SimpleDelegator
  def [](*args)
    super + 1
  end
end

SuperArray.new([1])[0] #=> 2
```

### 动态修改委派对象

下面的例子演示如何动态的修改接受委托的对象：

```ruby
# example 1:
class Stats
  def initialize
    @source = SimpleDelegator.new([])
  end

  def stats(records)
    @source.__setobj__(records)

    "Elements:  #{@source.size}\n" +
    " Non-Nil:  #{@source.compact.size}\n" +
    "  Unique:  #{@source.uniq.size}\n"
  end
end

s = Stats.new
puts s.stats(%w{James Edward Gray II})
puts
puts s.stats([1, 2, 3, nil, 4, 5, 1, 2])
```

另一个演示如何动态的修改接受委托的对象的例子：

```ruby
# example 2:
require 'delegate'

class TicketSeller
  def sellTicket
    return 'Here is a ticket'
  end
end

class NoTicketSeller
  def sellTicket
    "Sorry-come back tomorrow"
   end
end

class TicketOffice < SimpleDelegator
  def initialize
    @seller = TicketSeller.new
    @noseller = NoTicketSeller.new
    super(@seller)
  end

  def allowSales(allow = true)
    __setobj__(allow ? @seller : @noseller)
    allow
  end
end
```

```ruby
to = TicketOffice.new
to.sellTicket #=>  "Here is a ticket"
to.allowSales(false) #=> false
to.sellTicket #=> "Sorry-come back tomorrow"
to.allowSales(true) #=> true
to.sellTicket #=> "Here is a ticket"
```

### 相关链接

* [When to use Ruby DelegateClass instead of SimpleDelegator?](http://stackoverflow.com/questions/13104942/when-to-use-ruby-delegateclass-instead-of-simpledelegator-delegateclass-method)
* [Ruby 中文文档](http://www.kuqin.com/rubycndocument/man/addlib/delegate.html#SimpleDelegator)
