---
layout: post
title:  "Ruby 中的 eval 与 binding"
date:   2016-09-19 01:00:00
categories: ruby
tags: advanced
author: "Victor"
---

## 内部函数

> The Kernel module is included by class Object, so its methods are available in every Ruby object.

严格说来 Ruby 中没有函数。但我们可以把 Kernel 模块中定义的方法当作在任何地方都能用做函数。但正因为如此，当你需要重新定义这些方法的时候，就要考虑它对其它对象的影响。

## eval

## 基本用法

由于太多书都讲过 `eval` 的危害，所以有些工作了多年的程序员都可能完全没在自己的代码中出现过这个东西。比如我，甚至忘了 Ruby 中有这个方法。

```ruby
eval(string [, binding [, filename [,lineno]]]) → obj
# Evaluates the Ruby expression(s) in string. If binding is given, which must be a Binding object, the evaluation is performed in its context. If the optional filename and lineno parameters are present, they will be used when reporting syntax errors.
```

`eval` 把字符串 expr 当作 Ruby 程序来运行并返回其结果。乍看之下，`eval` 方法同在字符串中嵌入 `#{}` 的作用一样：

```ruby
str = "hello"
eval "str + ' Fred'" #=> "hello Fred"
"#{str} Fred" #=> "hello Fred"
```

但是，有些时候，结果却并非想象的那样，考虑下面的例子：

```ruby
exp = gets().chomp()   
puts(eval(exp)) #=> 8
puts("#{exp}") #=> 2*4
puts("#{eval(exp)}") #=> 8  
```

键入 2*4，发现两个方法给出的结果并不相同。这是因为，通过 `gets()` 方法接收到的是一个字符串，`#{}` 将它当成字符串处理，而不是一个表达式，但是 `eval(exp)` 将它作为一个表达式处理。

`eval` 会默认在当前上下文环境中执行。我们可以通过eval方法的第二个参数指明 `eval` 所运行代码的上下文环境，这个参数可以是 Binding 类对象或 Proc 类对象。

### Proc 和 Binding

Proc 类我[这里](/ruby/ruby-proc/)有讲过。

Binding 类的实例对象，可以封装代码在某一环境运行时的上下文，可以供以后使用。变量、方法、self 的值以及任何迭代器代码块都会被保留在该实例对象中。我们可以通过 `Kernel#binding` 方法来创建一个 Binding 类的实例对象。

```ruby
def foo
  a = 1
  binding
end

eval("p a", foo) # => 1
```

```ruby
class Demo
  def initialize(n)
    @secret = n
  end
  def get_binding
    binding
  end
end

k1 = Demo.new(99)
b1 = k1.get_binding
k2 = Demo.new(-3)
b2 = k2.get_binding

eval("@secret", b1)   #=> 99
eval("@secret", b2)   #=> -3
eval("@secret")       #=> nil
```

另外若指定了 fname 和 lineno 的话，将假定字符串位于 fname 文件 lineno 行，并且进行编译。这时可以显示栈跟踪(stack trace)等信息。

### 特殊类型

`eval` 还有几个方法变体：`instance_eval`, `module_eval`, `class_eval`。有兴趣的话可以读一下，相关链接中的 *Ruby 动态编程* 那一篇。

### 实际使用

有时候我们需要在日志中，输出全部一些变量信息：

```ruby
# config/initializers/logger_additions.rb
logger = ActiveRecord::Base.logger
def logger.debug_variables(bind)
  vars = eval('local_variables + instance_variables', bind)
  vars.each do |var|
    debug  "#{var} = #{eval(var, bind).inspect}"
  end
end
```

```ruby
# models/product.rb
logger.debug_variables(binding)
```

## 相关链接

* [内部函数](http://www.kuqin.com/rubycndocument/man/stdlib_function.html)
* [Binding](http://ruby-doc.org/core-2.2.3/Binding.html)
* [Ruby 动态编程](http://www.iteye.com/topic/375531)
* [Logging Variables](http://railscasts.com/episodes/86-logging-variables)
