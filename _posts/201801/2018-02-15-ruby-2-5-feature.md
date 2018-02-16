---
layout: post
title:  "Ruby 2.5 特性"
date:   2018-02-15 23:12:00

categories: ruby
tags: advanced
author: "Victor"
---

新年的鞭炮声伴随妈妈沉睡的轻鼾，很开心。新年快乐。

## 新方法

### `yield_self`
这应该是最重要的一个新方法，所以在本文中篇幅最长，放在最前面了。`yield_self` 是 `Kernel` 的一部分，所以你可以在任何对象上调用该方法。

`yield_self` 会让接受者执行给定代码块的结果，并返回代码块中最后的执行结果。

```ruby
"Hello".yield_self { |str| str + " World" }
#=> "Hello World"
```

它跟 Rails 中的 `try` 方法很像。不过 `try` 方法的当接受者是 `nil` 的时候总是返回 `nil`，而 `yield_self` 仍然返回代码块的执行结果。

还需要注意的是 `try` Rails 框架中的方法，并不是 Ruby 内置的。

```ruby
nil.yield_self { |str| str + "Hello World" }
#=> "Hello World"

nil.try { |obj| "Hello World" }
#=> nil
```

和 `tap` 的区别也十分明显，`tap` 总是返回接受者自身。

```ruby
"Hello".tap { |str| str + " World" }
#=> "Hello"
```

### 使用场景
在使用携带 block 的方法链时，`yield_self` 可以增强可读性，远远胜过使用嵌套的函数调用（nested function calls）。

```ruby
add_greeting = -> (str) { "HELLO " + str }
to_upper = -> (str) { str.upcase }

# with new `yield_self`
"world".yield_self(&to_upper).yield_self(&add_greeting)
#=> "HELLO WORLD"

# nested function calls
add_greeting.call(to_upper.call("world"))
#=> "HELLO WORLD"
```

另一个例子是 `yield_self` 可以让我们的代码执行逻辑更清楚。

```ruby
CSV.parse(File.read(File.expand_path("data.csv"), __dir__))
   .map { |row| row[1].to_i }
   .sum

# `yield_self`   
"data.csv"
  .yield_self { |name| File.expand_path(name, __dir__) }
  .yield_self { |path| File.read(path) }
  .yield_self { |body| CSV.parse(body) }
  .map        { |row|  row[1].to_i }
  .sum
```

不需要再写一堆的 `if` 语句了。

```ruby
events = Event.upcoming
events = events.limit(params[:limit])          if params[:limit]
events = events.where(status: params[:status]) if params[:status]
events

# `yield_self`   
Event.upcoming
  .yield_self { |events| params[:limit]  ? events.limit(params[:limit]) : events }
  .yield_self { |events| params[:status] ? events.where(status: status) : events }
```

最后看一个终极例子。

```ruby
# NORMAL
"https://api.github.com/repos/rails/rails"
  .yield_self { |it| URI.parse(it) }
  .yield_self { |it| Net::HTTP.get(it) }
  .yield_self { |it| JSON.parse(it) }
  .yield_self { |it| it.fetch("stargazers_count") }
  .yield_self { |it| "Rails has #{it} stargazers" }
  .yield_self { |it| puts it }

# GOOD
"https://api.github.com/repos/rails/rails"
  .yield_self { |url| URI.parse(url) }
  .yield_self { |url| Net::HTTP.get(url) }
  .yield_self { |response| JSON.parse(response) }
  .yield_self { |repo| repo.fetch("stargazers_count") }
  .yield_self { |stargazers| "Rails has #{stargazers} stargazers" }
  .yield_self { |string| puts string }

# BAD
"https://api.github.com/repos/rails/rails"
  .yield_self(&URI.method(:parse))
  .yield_self(&Net::HTTP.method(:get))
  .yield_self(&JSON.method(:parse))
  .yield_self { |_| _.fetch("stargazers_count") }
  .yield_self { |_| "Rails has #{_} stargazers" }
  .yield_self(&method(:puts))  
```

### Array#prepend and Array#append

有一对从 Rails 框架中引入的方法名。以前我们使用 `Array#unshift` 往数组内的头部插入数据，使用 `Array#push` 在数组最后插入一个数据。难以理解。

现在这两个方法分别改名为 `prepend` 和 `append`。完美，终于和其它语言一致了。


## added Hash#slice method

很方便的从 hash 中取出指定的 keys 对应的 values。

```ruby
blog = { id: 1, name: 'Ruby 2.5', description: 'BigBinary Blog' }
blog.slice(:name, :description)
#=> {:name=>"Ruby 2.5", :description=>"BigBinary Blog"}
```

## added delete_prefix and delete_suffix methods

过去要拿到一个命名空间下的控制器名称很麻烦。

```ruby
# 2.4
"Projects::CategoriesController".chomp("Controller")
#=> "Projects::Categories"

"Projects::CategoriesController".sub(/Projects::/, '')
#=> "CategoriesController"
```

现在

```ruby
# 2.5
"Projects::CategoriesController".delete_prefix("Projects::")
#=> "CategoriesController"

"Projects::CategoriesController".delete_suffix("Controller")
#=> "Projects::Categories"
```

## introduces Dir.children and Dir.each_child

以前用 `Dir.entries` 和 `Dir.foreach` 遍历的时候会把 `.` 和 `..` 也显示出来。

```ruby
Dir.entries("/Users/john/Desktop/test")
#=> [".", "..", ".config", "program.rb", "group.txt"]

Dir.foreach("/Users/john/Desktop/test") { |child| puts child }
.
..
.config
program.rb
group.txt
test2
```

现在可以改用 `Dir.children` 和 `Dir.each_child`，以及类似的 `Pathname#children` 和 `Pathname#each_child`。

```ruby
Dir.children("/Users/mohitnatoo/Desktop/test")
#=> [".config", "program.rb", "group.txt"]

Dir.each_child("/Users/john/Desktop/test") { |child| puts child }
.config
program.rb
group.txt
test2
```

### adds Hash#transform_keys method

引入了两个 Rails 的方法。`Hash#transform_values` 和 `Hash#transform_keys`，方便我们操作 key 和 value。

```ruby
h = { name: "John", email: "john@example.com" }
h.transform_keys { |k| k.to_s }
#=> {"name"=>"John", "email"=>"john@example.com"}
```

## 原方法引入新特性

### enumerable predicates accept pattern argument

Ruby 本身提供一系列的断言方法 `all?, none?, one?, any?`。过去我们要在这些断言方法后面跟随 block，并在其中进行条件判断。现在你可以直接传入一个表达式参数。这样在迭代的内部，会为每个对象依次调用 `===` 方法。

```ruby
if queries.any?(/LEFT OUTER JOIN/i)
  logger.log "Left outer join detected"
end

# Translates to:
queries.any? { |sql| /LEFT OUTER JOIN/i === sql }
```

### allows creating structs with keyword arguments

现在创建结构的时候只要最后一个参数是 `keyword_init: true`，那么当你实例化结构体的时候，就可以使用可变长度的关键字参数。

```ruby
Customer = Struct.new(:name, :email, keyword_init: true)
Customer.new(name: "John", email: "john@example.com")
Customer.new(name: "Victor")
```

## 改进

## allows rescue/else/ensure inside do/end blocks

```ruby
irb> array_from_user = [4, 2, 0, 1]
  => [4, 2, 0, 1]

irb> array_from_user.each do |number|
irb>   p 10 / number
irb> rescue ZeroDivisionError => exception
irb>   p exception
irb>   next
irb> end

# ruby 2.4
SyntaxError: (irb):4: syntax error, unexpected keyword_rescue,
expecting keyword_end
rescue ZeroDivisionError => exceptio

# ruby 2.5
2
5
#<ZeroDivisionError: divided by 0>
10
 => [4, 2, 0, 1]
 ```

### removed top level constant lookup

Ruby 2.5 throws an error if it is unable to find constant in the specified scope.

```ruby
irb> class Project
irb> end
=> nil

irb> class Category
irb> end
=> nil

irb> Project::Category

# ruby 2.4
(irb):5: warning: toplevel constant Category referenced by Project::Category
 => Category

# ruby 2.5
NameError: uninitialized constant Project::Category Did you mean? Category from (irb):5
```

### requires pp by default

可以在控制台中直接使用 `pp` 命令了。

## 参考

* [Ruby 2.5](http://blog.bigbinary.com/categories/Ruby-2-5)
* [yield_self in Ruby 2.5](https://mlomnicki.com/yield-self-in-ruby-25/)
