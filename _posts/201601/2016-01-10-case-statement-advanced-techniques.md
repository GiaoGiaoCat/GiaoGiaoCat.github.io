---
layout: post
title:  "case 表达式的高级技巧"
date:   2016-01-10 14:00:00
categories: ruby
tags: tip advanced
author: "Victor"
---

通常我们对 case 表达式的理解是用它来替换一大堆的 if 判断，那么它还有啥技巧呢？

```ruby
case "Hi there"
when String
  puts "case statements match class"
end

# outputs: "case statements match class"
```

因为 Ruby 的 case 表达式内部使用的是 `===` 进行条件判断，所以先来一起复习下 `===` 的用法。

### `===` 运算符简介

可以先复习一下[这里](/ruby/codecademy-ruby/)，

* 如果是类来调用 `===` 方法，就会比较另一个对象是否为当前类的实例
* 如果是对象调用 `===` 方法，和 `==` 用法相同，仅比较两个值是否相等

```ruby
# Here, the Class.===(item) method is called, which returns true if item is an instance of the class

String === "hello" # true
String === 1 # false
```

字符串，范围，正则表达式都定义了自己的 `===(item)` 方法，它们的工作方式也和我们期待的一样。我们也可以给自己定义的类增加 `===(item)` 方法。

### 使用范围进行匹配验证

`range === n` 等于 `range.include?(n)`。

```ruby
case 5
when (1..10)
  puts "case statements match inclusion in a range"
end

# outputs "case statements match inclusion in a range"
```

### 使用正则表达式进行匹配验证

`/regexp/ === "string"`

```ruby
case "FOOBAR"
when /BAR$/
  puts "they can match regular expressions!"
end

# outputs "they can match regular expressions!"
```

### 使用 `procs` 和 `lambdas` 进行匹配验证

这里需要注意的是 `Proc#===(item)` 等价于 `Proc#call(item)`。这意味着你可以使用 `procs` 和 `lambdas` 进行动态的匹配。

```ruby
case 40
when -> (n) { n.to_s == "40" }
  puts "lambdas!"
end

# outputs "lambdas"
```

### 创建自己的匹配类

正如前文提到的，我们可以通过在类里面添加 `===` 方法，创建自己的匹配类。这样的好处是可以拆分我们的业务逻辑到多个细小的类。

```ruby
class Success
  def self.===(item)
    item.status >= 200 && item.status < 300
  end
end

class Empty
  def self.===(item)
    item.response_size == 0
  end
end

case http_response
when Empty
  puts "response was empty"
when Success
  puts "response was a success"
end
```
