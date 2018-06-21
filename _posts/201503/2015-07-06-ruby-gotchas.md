---
layout: post
title:  "Ruby 中那些你绕不过的「坑」"
date:   2015-07-06 16:30:00
categories: ruby
tags: advanced
author: "Victor"
---

## `and 和 or` 不同于 `&& 和 ||`

```ruby
surprise = true and false # => surprise is true
surprise = true && false  # => surprise is false
```

### 最佳实践

只使用 `&& 和 ||` 运算符。

### 详情

* `and 和 or` 运算符的优先级比 `&& 和 ||` 低
* `and 和 or` 的优先级比 `=` 低，而 `&& 和 ||` 的优先级比 `=` 高
* `and 和 or` 的优先级相同，而 `&&` 的优先级比 `||` 高

我们来给上述示例代码加上括号，这样就可以明显地看出 `and` 和 `&&` 在用法上的不同之处了。

```ruby
(surprise = true) and false # => surprise is true
surprise = (true && false)  # => surprise is false
```

也有这样的说法：`and 和 or` 用于流程控制，而 `&& 和 ||` 用于布尔型运算。但我认为：不要使用这些运算符的关键字版本 `and / or / not`，而使用更为清晰的 `if` 和 `unless` 等。更加明确，更少困惑，更少 bugs。

延伸阅读：[Difference between “or” and || in Ruby?](http://stackoverflow.com/questions/2083112/difference-between-or-and-in-ruby)

## `eql?` 不同于 `==`（也不同于 `equal?` 或 `===`）

```ruby
1 == 1.0   # => true
1.eql? 1.0 # => false
```

### 最佳实践

只使用 `==` 运算符。

### 详情

`==、===、eql? 和 equal?` 都是互不相同的运算符，各自有不同的用法，分别用于不同的场合。当你要进行比较时，总是使用 `==`，除非你有特殊的需求（比如你真的需要区分 1.0 和 1）或者出于某些原因重写（override）了某个运算符。

没错，`eql?` 可能看起来要比平凡的 `==` 更为聪明，但是你真的需要这样吗，去区分两个相同的东西？

延伸阅读：[What’s the difference between equal?, eql?, ===, and ==?](http://stackoverflow.com/questions/7156955/whats-the-difference-between-equal-eql-and)

## `super` 不同于 `super()`

```ruby
class Foo
  def show
    puts 'Foo#show'
  end
end

class Bar < Foo
  def show(text)
    super

    puts text
  end
end

Bar.new.show('test') # ArgumentError: wrong number of arguments (1 for 0)
```

### 最佳实践

在这里，省略括号可不仅仅是品味（或约定）的问题，而是确实会影响代码的逻辑。

### 详情

使用 `super`（没有括号）调用父类方法时，会将传给这个方法的参数原封不动地传给父类方法（因此在 `Bar#show` 里面的 `super` 会变成 `super('test')`，引发了错误，因为父类的方法不接收参数）
`super()`（带括号）在调用父类方法时不带任何参数，正如我们期待的那样。
延伸阅读：[Super keyword in Ruby](http://stackoverflow.com/questions/4632224/super-keyword-in-ruby)

## 自定义异常不能继承 Exception

```ruby
class MyException < Exception
end

begin
  raise MyException
rescue
  puts 'Caught it!'
end

# MyException: MyException
#       from (irb):17
#       from /Users/karol/.rbenv/versions/2.1.0/bin/irb:11:in `<main>'
```


（上述代码不会捕捉到 `MyException`，也不会显示 'Caught it!' 的消息。）

### 最佳实践

* 自定义异常类时，继承 `StandardError` 或任何其后代子类（越精确越好）。
* 永远不要直接继承 `Exception`。
* 永远不要 `rescue Exception`。如果你想要大范围捕捉异常，直接使用空的 `rescue` 语句（或者使用 `rescue => e` 来访问错误对象）。

### 详情

当你使用空的 `rescue` 语句时，它会捕捉所有继承自 `StandardError` 的异常，而不是 `Exception`。
如果你使用了 `rescue Exception`（当然你不应该这样），你会捕捉到你无法恢复的错误（比如内存溢出错误）。而且，你会捕捉到 SIGTERM 这样的系统信号，导致你无法使用 CTRL+C 来中止你的脚本。

延伸阅读：[Why is it bad style to `rescue Exception => e` in Ruby?](http://stackoverflow.com/questions/10048173/why-is-it-bad-style-to-rescue-exception-e-in-ruby)

## `class Foo::Bar` 不同于 `module Foo; class Bar`

```ruby
MY_SCOPE = 'Global'

module Foo
  MY_SCOPE = 'Foo Module'

  class Bar
    def scope1
      puts MY_SCOPE
    end
  end
end

class Foo::Bar
  def scope2
    puts MY_SCOPE
  end
end

Foo::Bar.new.scope1 # => "Foo Module"
Foo::Bar.new.scope2 # => "Global"
```

### 最佳实践

总是使用长的，更清晰的，`module` 把 `class` 包围的写法：

```ruby
module Foo
  class Bar
  end
end
```

### 详情

* `module` 关键字（`class` 和 `def` 也一样）会对其包围的区域创建新的词法作用域（lexical scope）。所以，上面的 `module Foo` 创建了 'Foo' 作用域，常量 `MY_SCOPE` 和它的值 'Foo Module' 就在其中。
* 在这个 `module` 中，我们声明了 `class Bar`，又会创建新的词法作用域（名为 'Foo::Bar'），它能够访问父作用域（'Foo'）和定义在其中的所有常量。
* 然而，当你使用了这个 `::` 「捷径」来声明 `Foo::Bar` 时，`class Foo::Bar` 又创建了一个新的词法作用域，名字也叫 'Foo::Bar'，但它没有父作用域，因此不能访问 'Foo' 里面的东西。
* 因此，在 `class Foo::Bar` 中我们只能访问定义在脚本的开头的 `MY_SCOPE` 常量（不在任何 `module` 中），其值为 'Global'。

延伸阅读：[Ruby – Lexical scope vs Inheritance](http://stackoverflow.com/questions/15119724/ruby-lexical-scope-vs-inheritance)

## 多数 `bang!` 方法如果什么都没做就会返回 `nil`

```ruby
'foo'.upcase! # => "FOO"
'FOO'.upcase! # => nil
```

### 最佳实践

永远不要依赖于内建的 `bang!` 方法的返回值，比如在条件语句或流程控制中：

```ruby
@name.upcase! and render :show
```

上面的代码会造成一些无法预测的行为（或者更准备地说，我们可以预测到当 `@name` 已经是全部大写的时候就会失败）。另外，这个示例也再一次说明了为什么你不应该使用 `and/or` 来控制流程。敲两个回车吧，不会有树被砍的。

```ruby
@name.upcase!
render :show
```

## `attribute=(value)` 方法永远返回传给它的 value 而无视 `return` 值

```ruby
class Foo
  def self.bar=(value)
    @foo = value

    return 'OK'
  end
end

Foo.bar = 3 # => 3
```

（注意这个赋值方法 bar= 返回了 3，尽管我们显式地在最后 return 'OK'。）

### 最佳实践

永远不要依赖赋值方法的返回值，比如下面的条件语句：

```ruby
puts 'Assigned' if (Foo.bar = 3) == 'OK' # => nil
```

显然这个语句不会如你所想。

延伸阅读：[ruby, define []= operator, why can’t control return value?](http://stackoverflow.com/questions/6366834/ruby-define-operator-why-cant-control-return-value)

## `private` 并不会让你的 `self.method` 成为私有方法

```ruby
class Foo

  private
  def self.bar
    puts 'Not-so-private class method called'
  end

end

Foo.bar # => "Not-so-private class method called"
```

（注意，如果这个方法真的是私有方法，那么 Foo.bar 就会抛出 `NoMethodError`。）

### 最佳实践

要让你的类方法变得私有，你需要使用 `private_class_method :method_name` 或者把你的私有类方法放到 `class << self block` 中：


```ruby
class Foo

  class << self
    private
    def bar
      puts 'Class method called'
    end
  end

  def self.baz
    puts 'Another class method called'
  end
  private_class_method :baz

end

Foo.bar # => NoMethodError: private method `bar' called for Foo:Class
Foo.baz # => NoMethodError: private method `baz' called for Foo:Class
```

延伸阅读：[creating private class method](http://stackoverflow.com/questions/4952980/creating-private-class-method)

## 原文

* [Ruby 中那些你绕不过的「坑」](http://zhaowen.me/blog/2014/03/04/ruby-gotchas/)

