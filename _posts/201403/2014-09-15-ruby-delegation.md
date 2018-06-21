---
layout: post
title:  "Ruby 中的委托"
date:   2014-09-15 11:30:00
categories: ruby
tags: metaprogramming
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

```DelegateClass``` 接受一个类作为参数。

### 基本用法

```ruby
class MyClass < DelegateClass(ClassToDelegateTo) # Step 1
  def initialize
    super(obj_of_ClassToDelegateTo) # Step 2
  end
end
```
### 例子

```ruby
require 'delegate'  

class MyQueue < DelegateClass(Array)  

  def initialize(arg=[])  
    super(arg)  
  end  

  alias_method :enqueue, :push  
  alias_method :dequeue, :shift  
end  
```

```ruby
mq = MyQueue.new  
mq.enqueue(123)  
mq.enqueue(234)  
p mq.dequeue #=> 123
p mq.dequeue #=> 234  
```

另外一个例子。

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
__setobj__ # 将接受委托的对象变为 obj。
```

请注意，因为只有在生成时才会进行委托方法的定义，所以即使接受委托的对象和 obj 之间存在实例方法上的差异，也无法再次设定委托实例方法。因此，这里的建议是新的委托对象最好和原始的委托对象保持类型一致。

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

### Forwarding 和 SingleForwardable

如果你想要更多的控制，你可能想做到方法的委托，而不是类的委托，这时你可以使用 ```forwardable``` 库。

### 例子

```ruby
require 'forwardable'  

class MyQueue  
  extend Forwardable  

  def initialize(obj=[])  
    @queue = obj # delegate to this object  
  end  

  def_delegator :@queue, :push,  :enqueue  
  def_delegator :@queue, :shift, :dequeue  

  def_delegators :@queue, :clear, :empty?, :length, :size, :<<  

  # Any additional stuff...  
end  
```

在这里要注意，```def_delegator``` 方法也就是把 ```push``` 方法委托到 ```@queue``` 对象，并且重命名为 ```enqueue``` 方法。
而 ```def_delegators``` 则是直接把他后面的参数都委托给 ```@queue``` 对象，并且方法名不变。

```ruby
q1 = MyQueue.new                    # use an array  
q2 = MyQueue.new(my_array)          # use one specific array  
q3 = MyQueue.new(Queue.new)         # use a Queue (thread.rb)  
q4 = MyQueue.new(SizedQueue.new)    # use a SizedQueue (thread.rb)  
```

q3 和 q4 都是线程安全的，因为他们委托给了一个线程安全的对象。

```SingleForwardable``` 操作实例。当你需要把一个指定的对象委托给另一个对象时，就用得到了。

### 相关链接

* [When to use Ruby DelegateClass instead of SimpleDelegator?](http://stackoverflow.com/questions/13104942/when-to-use-ruby-delegateclass-instead-of-simpledelegator-delegateclass-method)
* [Ruby 中文文档](http://www.kuqin.com/rubycndocument/man/addlib/delegate.html#SimpleDelegator)
