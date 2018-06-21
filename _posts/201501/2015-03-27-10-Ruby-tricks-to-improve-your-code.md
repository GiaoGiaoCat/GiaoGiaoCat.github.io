---
layout: post
title:  "10 Ruby tricks to improve your code (or not)"
date:   2015-03-27 12:20:00
categories: ruby
tags: tip
author: "Victor"
---

本文介绍 10 个你可能早就知道的小技巧。

### 1. Create a hash from a list of values

```ruby
Hash['key1', 'value1', 'key2', 'value2']

# => {"key1"=>"value1", "key2"=>"value2"}
```

### 2. Lambda Literal `->`

`->` 是新引入的一个特性，可以很方便的创建 Lambda 表达式

```ruby
a = -> { 1 + 1 }
a.call
# => 2

a = -> (v) { v + 1 }
a.call(2)
# => 3
```

### 3. Double star `**`

`**` 是一个有趣的语法糖。`a` 是一个常规的参数；`*b` 会把传进来的第一个参数之后的参数压入一个数组；`**c` 会把传递进来的参数中最后面的部分，按照 key 和 value 的形式压入 hash。

```ruby
def my_method(a, *b, **c)
  return a, b, c
end

my_method(1)
# => [1, [], {}]

my_method(1, 2, 3, 4)
# => [1, [2, 3, 4], {}]

my_method(1, 2, 3, 4, a: 1, b: 2)
# => [1, [2, 3, 4], {:a=>1, :b=>2}]
```

### 4. Handle single object and array in the same way

有时候你的变量可能接受一个单一对象或一个数组对象。为了避免检查接受对象类型，你可以使用 `[*something]` 或 `Array(something)`。

```ruby
stuff = 1
stuff_arr = [1, 2, 3]

[*stuff].each { |s| s }
[*stuff_arr].each { |s| s }

Array(stuff).each { |s| s }
Array(stuff_arr).each { |s| s }
```

### 5. Double Pipe Equals `||=`

`||=` 的意思如下

```ruby
a || a = b # Correct
```

而不是

```ruby
a = a || b # Wrong
```

下面是一个典型的场景

```ruby
def total
  @total ||= (1..100000000).to_a.inject(:+)
end
```

现在你可以在其他方法中调用 `total` 方法，并且只会在第一次调用时进行计算。

### 6. Mandatory hash parameters

强制关键字参数是 Ruby 2.0 引入的一个新特性，过去我们要定义一个可接受 hash 作为参数的方法要这样：

```ruby
def my_method({})
end
```

现在我们可以指定参数中能接受的参数：

```ruby
def my_method(a:, b:, c: 'default')
  return a, b, c
end

my_method(a: 1)
# => ArgumentError: missing keyword: b

my_method(a: 1, b: 2)
# => [1, 2, "default"]

my_method(a: 1, b: 2, c: 3)
# => [1, 2, 3]

hash = { a: 1, b: 2, c: 3 }
my_method(hash)
# => [1, 2, 3]
```

### 7. Generate array of alphabet or numbers

```ruby
('a'..'z').to_a
# => ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"]

(1..10).to_a
# => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

### 8. Tap

`tap` 是提高代码可读性的一个小方法，以下面的类来举例：

```ruby
def User
  attr_accessor :a, :b, :c
end
```

比如说我们要实例化一个 user 并且让为它设置属性：

```ruby
def my_method
  o = User.new
  o.a = 1
  o.b = 2
  o.c = 3
  o
end
```

你可是使用 `tap` 来实现同样的功能：

```ruby
def my_method
  User.new.tap do |o|
    o.a = 1
    o.b = 2
    o.c = 3
  end
end
```

`tap` 把调用该方法的对象传递给代码块，并返回结果。

Basically, the tap method yields the calling object to the block and returns it.

### 9. Default value for hash (Bad trick)

默认情况下，当你访问 hash 中不存在的键，则会得到 `nil`。我们可以改变 hash 默认的初始化动作。

```ruby
a = Hash.new(0)
a[:a]
# => 0

a = Hash.new({})
a[:a]
# => {}

a = Hash.new('lolcat')
a[:a]
# => "lolcat"
```

注意，除非你很明确的知道这个方法可能带来的破坏。否则，请尽量不要用这种写法。因为你一旦为 hash 设置了默认值，那么该 hash 中的其它 key 都会引用相同的默认值对象。

```ruby
[1] pry(main)> a = Hash.new([])
=> {}
[2] pry(main)> a[:a]
=> []
[3] pry(main)> a[:a] << 1
=> [1]
[4] pry(main)> a[:a]
=> [1]
[5] pry(main)> a[:b]
=> [1]
[6] pry(main)> a[:b] << 2
=> [1, 2]
[7] pry(main)> a[:b]
=> [1, 2]
[8] pry(main)> a[:a]
=> [1, 2]
```

### 10. heredocs

`EOT` 会破坏代码缩进风格：

```ruby
def my_method
  <<-EOT
Some
Very
Interesting
Stuff
  EOT
end
```

有个小技巧来避免这种情况，通过使用 gsub 配合正则来删除前面的，保持缩进一致。

```ruby
def my_method
  <<-EOT.gsub(/^\s+/, '')
    Some
    Very
    Interesting
    Stuff
  EOT
end
```

### 原文

http://samurails.com/ruby/ruby-tricks-improve-code/
