---
layout: post
title:  "Ruby 基础教程第4版读书笔记 2"
date:   2014-11-12 09:40:00
categories: ruby
tags: learningnote
author: "Victor"
---

## Ruby 的基础

### 运算符

赋值运算符

```
&&=, ||=, ^=, &=, |=, <<=, >>=
+=, -=, *=, /=, %=, **=,
```

```ruby
$stdin.lineno += 1
$stdin.lineno = $stdin.lineno + 1
```

* 表达式的执行顺序是从左到右
* 如果逻辑表达式的真假已经可以确定，则不会再判断剩余的表达式
* 最后一个表达式的值为整体逻辑表达式的值

```ruby
item = nil
if ary
  item = ary[0]
end

item = ary && ary[0]
```

只有在 var 为 **nil** 或者 **false** 的时候，才把 1 赋值给它。这是给变量定义默认值的常用写法。

```ruby
var = var || 1

var ||= 1
```

* ```x..y``` 和 ```x...y``` 的区别是前者包含y，后者不包含y。
* 如果数值以外的对象也实现了根据当前值生成下一个值的方法，那么通过指定范围的起点和终点就可以生成 **Range** 对象。
* 在 Range 对象内部，可以使用 ```succ``` 方法根据起点值逐个生成接下来的值。具体的说，就是对 ```succ``` 方法的返回值调用 ```succ``` 方法，然后再循环，直到得到的值比终点值大时才结束。


运算符的优先级由高到低如下

```
::
[]
+(一元运算符) ! ~
**
-(一元运算符)
* / %
+ -
<< >>
&
| ^
> >= < <=
<=> == === != =~ !~
&&
||
?:(条件运算符)
.. ...
=(其它赋值运算符)
not
and or
```

Ruby 的运算符大多都是作为实例方法提供给我们使用的，除了下面的方法之外我们都可以很方便的定义或者重定义运算符，改变其原有的含义。

```
::, &&, ||, .., ..., ?:, not, =, and, or
```

定义二元运算符时，我们常把参数名定义为 other。
另外在定义方法时候，使用 ```self.class``` 可以灵活的处理继承和 Mix-in。

```ruby
class Point
  attr_reader :x, :y

  def initialize(x = 0, y = 0)
    @x, @y = x, y
  end

  def inspect
    "#{@x} #{@y}"
  end

  def +(other)
    self.class.new(x + other.x, y + other.y)
  end

  def -(other)
    self.class.new(x - other.x, y - other.y)
  end
end

point0 = Point.new(3, 6)
point1 = Point.new(1, 8)

p point0 + point1
```

可定义的一元运算符有 ```+, -, ~, !``` 4个。它们分别以 ```+@, -@, ~@, !@``` 为方法名进行方法的定义。

```ruby
class Point
  def +@
    dup # 返回自己的副本
  end

  def -@
    self.class.new(-x, -y) # 颠倒x, y各自的正负
  end

  def ~@
    self.class.new(-y, x) # 坐标翻转90度
  end
end

point = Point.new(3, 6)
p +point
p ~point
```

数组，散列中的 obj[i], obj[i] = x 这样的方法，称为下标方法。定义下标时的方法名分别为 ```[]``` 和 ```[]=```

```ruby
class Point
  def [](index)
    case index
    when 0 then x
    when 1 then y
    else
      raise ArgumentError, "out of range `#{index}'"
    end
  end

  def []=(index, val)
    case index
    when 0 then self.x = val
    when 1 then self.y = val
    else
      raise ArgumentError, "out of range `#{index}'"
    end
  end
end

point = Point.new(3, 6)
point[0]
point[1] = 2
```

### 错误处理与异常

对于可预期的错误，我们需要留意一下两点：

* 是否破坏了输入的数据，特别是人工制作的数据。
* 是否可以对错误的内容及原因做出相应的提示。

异常处理有以下优点:

* 程序不需要逐个确认处理结果，也能自动检查出程序错误
* 会同时报告发生错误的位置，便于排查错误
* 正常处理与错误处理的程序可以分开书写，是程序便于阅
* 不捕捉异常意味着程序出问题就会马上终止

#### 异常处理的写法

```ruby
begin
  可能会发生异常的处理
rescue => 引用异常对象的变量
  发生异常时的处理
  sleep 10
  retry
ensure
  不管是否发生异常都希望执行的处理
end
```

* 即使不指定变量名，Ruby 会把异常对象赋值给变量 **$!**
* ```ensure``` 中的处理，在程序跳出 ```begin ~ end``` 部分时一定会被执行。
* 在 ```resure``` 中使用 ```retry``` 后， begin 以下的处理会重做一遍，注意的是这样容易形成死循环。

#### 指定需要捕捉的异常

当存在多个种类的异常，且需要按异常种类分别进行处理时，我们可以用多个 ```resuce``` 来分开处理。

```ruby
begin
  可能会发生异常的处理
rescue Exception1, Exception2 => 变量
  发生异常时的处理
rescue Exception3 => 变量
  发生异常时的处理
rescue
  发生异常时的处理
end
```

* Ruby 中所有的异常都是 ```Exception``` 类的子类。
* ```rescue``` 中不指定异常类时，程序会默认捕捉 ```StandardError``` 类及其子类。
* 我们在自定义异常时，一般先定义继承 ```StandardError``` 类的新类，在继承这个新类。

#### 主动抛出异常

使用 ```raise``` 方法，可以使程序主动抛出异常。

* ```raise message``` 抛出 ```RuntimeError``` 异常，并把字符串 message 设置给新生成的异常对象
* ```raise 异常类``` 抛出指定的异常
* ```raise 异常类, message```
* ```raise``` 在 ```rescue``` 外抛出 ```RuntimeError``` 异常。在 ```rescue``` 调用时，会抛出最后一次发生的异常 **$!**
