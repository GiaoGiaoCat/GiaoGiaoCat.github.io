---
layout: post
title:  "Learning Ruby: class << self"
date:   2014-09-01 19:00:00
categories: ruby
tags: metaprogramming
author: "Victor"
---

```class << self``` 相当于批量定义了 ```def self.someMethods```

在ruby中经常可以见到这样的写法：

```ruby
class A
  class << self
     def hello
        puts "hello"
     end
   end
end
```

这样的写法和

```ruby
class A
  def self.hello
     puts "hello"
   end
end
```

可以说是完全一样的。第一种写法的一个好处是，如果需要在一个 class 或者 module 里面定义多个类级别的方法， 这种写法可以少写很多个 "self." :)

还有一个好处是，可以使用 ```attr_reader/attr_accessor``` 之类的 metaprogramming 的技巧。

这里还有另一个例子加深理解：

```ruby
module Lib
  class << self
    def fuc
      puts "in Lib'fuc"
      puts self
    end
  end
end

class A
  include Lib
  def self.fuc
    puts "in A'fuc"
    puts self
  end
end

Lib.fuc
A.fuc

gets
```

输出结果是

```
in Lib'fuc
Lib
in A'fuc
A
```

## 详解

### 从单例方法定义开始

首先 ```class << foo``` 表达式打开了 ```foo``` 的单例类，它允许你给指定的对象定义一些单例方法。

```ruby
a = 'foo'
class << a
  def inspect
    '"bar"'
  end
end
a.inspect   # => "bar"

a = 'foo'   # new object, new singleton class
a.inspect   # => "foo"
```

### 类和模块的 class << self

所以 ```class << self``` 的意思就是打开 ```self``` 的单例类，允许我们重新给 ```self``` 对象定义方法。当 ```self``` 在 class 或者 module 内的时候，我们就是定义 class/module ("static") 方法。

```ruby
class String
  class << self
    def value_of obj
      obj.to_s
    end
  end
end

String.value_of 42   # => "42"
```

也可以写成

```ruby
class String
  def self.value_of obj
    obj.to_s
  end
end
```

甚至更短的

```ruby
def String.value_of obj
  obj.to_s
end
```

### 函数内部的 class << self

当在一个函数内部，```self``` 用来引用调用该函数的对象。在下面的例子中 ```class << self``` 打开了单例类

```ruby
class StateMachineExample
  def process obj
    process_hook obj
  end

private
  def process_state_1 obj
    # ...
    class << self
      alias process_hook process_state_2
    end
  end

  def process_state_2 obj
    # ...
    class << self
      alias process_hook process_state_1
    end
  end

  # Set up initial state
  alias process_hook process_state_1
end
```

这里例子中，每一个 StateMachineExample 的实例都有一个 ```process_hook``` 方法被别名为 ```process_state_1```，但是要注意看接下来的动作。对应的实例方法可以重新定义 ```process_hook``` 将其别名为 ```process_state_2```。这个动作只影响自身，而不会影响 StateMachineExample 的其他实例。

所以每当有对象调用 ```process``` 方法 (该方法被我们重定义为 ```process_hook```)，该行为变化取决于对象自身是什么状态。

### 相关阅读

* [ruby 的 ```class << self```, ```class_eval``` 和 ```instance_eval``` 的区别](http://blog.csdn.net/lyx2007825/article/details/10089115)
* [一个创建闭包的小技巧](http://simohayha.iteye.com/blog/200304)
