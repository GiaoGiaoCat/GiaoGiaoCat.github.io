---
layout: post
title:  "Ruby 2 Keyword Arguments"
date:   2014-11-06 19:10:00
categories: ruby
tags: tip
author: "Victor"
---

## 参数不确定的方法

* ```*变量名``` 这种形式的参数，只能在方法定义的参数列表中出现一次。

```ruby
def foo(*args)
  args
end

p foo(1, 2, 3) #=> [1, 2, 3]
```

```ruby
def meth(arg, *args)
  [arg, args]
end

p meth(1) #=> [1, []]
p meth(1, 2, 3) #=> [1, [2, 3]]
```

```ruby
def a(a, *b, c)
  [a, b, c]
end

p a(1, 2, 3, 4) #=> [1, [2, 3], 4]
p a(1, 2) #=> [1, [], 3]
```

## 关键字参数

* 使用关键字参数，可以将参数名与参数值成对地传给方法内部使用。
* 如果把未定义的参数名传给方法，程序就会报错。

```ruby
def area(x: 0, y: 0, z: 0)
  (x * y + y * z + z * x) * 2
end

area(x: 2, y: 3, z: 4)
area(x: 2, z: 4, y: 3) # 改变参数顺序
area(x: 2, y: 3) # 省略参数
```

* 可以使用 ```**变量名``` 的形式来接收未定义的参数。

```ruby
def meth(x: 0, y: 0, z: 0, **args)
  [x, y, z, args]
end

# BAD code, not works.
# def meth(x: 0, y: 0, z: 0, args = {})
#   [x, y, z, args]
# end

p meth(z: 4, y: 3, x: 2) #=> [2, 3, 4, {}]
p meth(x: 2, z: 3, v: 4, w: 5) #=> [2, 0, 3, {:v => 4, :w => 5}]
```

* 关键字参数可以与普通参数搭配使用。

```ruby
def func(a, b: 1, c: 2)
end

func(1, b: 2, c: 3)
```

* 用散列传递参数

调用关键字参数定义的方法时，可以使用符号作为键的散列来传递参数。程序会自动检查散列的键与定义的参数名是否一致，并将与散列的键一致的参数名传递给方法。

```ruby
def area(x: 0, y: 0, z: 0)
end

args1 = {x: 2, y: 3, z: 4}
p area(args1)

args2 = {x: 2, z: 4}
p area(args2)
```

* 将数组分解成参数

在调用方法时，如果以 ```*数组``` 这样的形式指定参数，这时传递给方法的就不是数组本身，而是数组的各元素都被按照顺序传递给方法。但需要注意的是，数组元素个数需要和方法参数的数量一致。

```ruby
def foo(a, b, c)
 a + b + c
end

foo(1, 2, 3)
args = [2, 3]
foo(1, *args)
```

* 散列作为参数传递

将散列的字面量作为参数传递给方法时可以省略 ```{}```

```ruby
def foo(arg)
  arg
end

foo({"a" => 1, "b" => 2})
foo("a" => 1， "b" => 2)
foo(a: 1, b: 2)
```

当有多个参数，但只将散列作为最后一个参数传递给方法时，可以使用下面的写法：

```ruby
def bar(arg1, arg2)
  [arg1, arg2]
end

bar(100, {a: 1, b: 2}) #=> [100, {:a => 1, :b => 2}]
bar(100, a: 1, b: 2) #=> [100, {:a => 1, :b => 2}]
```

## 在代码块中使用关键字参数

```ruby
define_method(:foo) do |bar: 'default'|
  puts bar
end

foo # => 'default'
foo(bar: 'baz') # => 'baz'
```

## 结论

使用关键字参数定义方法，既可以对键进行限制，又可以定义参数的默认值。建议在实际编程的时候多尝试这个方法。

## 相关链接

* [Keyword arguments in Ruby 2.0](http://brainspec.com/blog/2012/10/08/keyword-arguments-ruby-2-0/)
* [Ruby 2 Keyword Arguments](http://robots.thoughtbot.com/ruby-2-keyword-arguments)
* [Ruby 2.0 : Keyword arguments](http://dev.af83.com/2013/02/18/ruby-2-0-keyword-arguments.html)
