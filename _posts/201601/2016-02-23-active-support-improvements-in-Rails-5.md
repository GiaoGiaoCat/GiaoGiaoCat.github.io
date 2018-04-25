---
layout: post
title:  "Active Support Improvements in Rails 5"
date:   2016-02-22 12:00:00
categories: rails
tags: rails5 time enumerable bigbinary
author: "Victor"
---

## Improvements in Date, Time and Datetime

参考[这个PR](https://github.com/rails/rails/pull/18335/files)，DHH 认为代码的可读性优先级高于DRY。所以增加了一堆 alias 方法。

### prev_day and next_day

没啥说的，就是给 `Date`, `Time` 和 `DateTime` 的 `#yesterday` 和 `#tomorrow` 增加了另外一个名称。

### same_time

Rails 4.x 的时候 `next_week` 和 `prev_week` 返回那个星期的开头时间。

```ruby
Time.current #=> Fri, 12 Feb 2016 08:53:31 UTC +00:00
Time.current.next_week #=> Mon, 15 Feb 2016 00:00:00 UTC +00:00
Time.current.next_week(:tuesday) #=> Tue, 16 Feb 2016 00:00:00 UTC +00:00
Time.current.prev_week(:tuesday) #=> Tue, 02 Feb 2016 00:00:00 UTC +00:00
```

现在利用 `same_time: true` 我们可以拿到上周三了。

```ruby
Time.current #=> Fri, 12 Feb 2016 09:15:10 UTC +00:00
Time.current.next_week #=> Mon, 15 Feb 2016 00:00:00 UTC +00:00
Time.current.next_week(same_time: true) #=> Mon, 15 Feb 2016 09:15:20 UTC +00:00
Time.current.prev_week #=> Mon, 01 Feb 2016 00:00:00 UTC +00:00
Time.current.prev_week(same_time: true) #=> Mon, 01 Feb 2016 09:16:50 UTC +00:00
```

### on_weekend?

顾名思义，当周六、日的时候返回 true

```ruby
Time.current #=> Fri, 12 Feb 2016 09:47:40 UTC +00:00
Time.current.on_weekend? #=> false
Time.current.tomorrow #=> Sat, 13 Feb 2016 09:48:47 UTC +00:00
Time.current.tomorrow.on_weekend? #=> true
```

### on_weekday?

与 `on_weekend?` 相反。工作日的时候返回 true

### next_weekday and prev_weekday

周五的时候 `next_weekday` 返回下周一

### Time.days_in_year

```ruby
# Gives number of days in current year, if year is not passed.
Time.days_in_year #=> 366

# Gives number of days in specified year, if year is passed.
Time.days_in_year(2015) #=> 365
```

## Improvements in Enumerable

### pluck

参考[这个PR](https://github.com/rails/rails/pull/20350)，现在只要是 Enumerable 对象都可以使用 `pluck` 方法了。

```ruby
users = [{id: 1, name: 'Max'}, {id: 2, name: 'Mark'}, {id: 3, name: 'George'}]
users.pluck(:name) #=> ["Max", "Mark", "George"]
# Takes multiple arguments as well
users.pluck(:id, :name) #=> [[1, "Max"], [2, "Mark"], [3, "George"]]
```

这对性能上的改进显而易见。当 ActiveRecord 的 relation 已经加载完毕，再次执行 `pluck` 动作的话，会直接调用 Enumerable 的 `pluck` 方法，而不会再次生成 SQL 查询。

```ruby
# With Rails 4.x
users = User.all
SELECT `users`.* FROM `users`

users.pluck(:id, :name)
SELECT "users"."id", "users"."name" FROM "users"

=> [[2, "Max"], [3, "Mark"], [4, "George"]]


# With Rails 5
users = User.all
SELECT "users".* FROM "users"

# does not fire any query
users.pluck(:id, :name)
=> [[1, "Max"], [2, "Mark"], [3, "George"]]
```

### without

没错，又是一个语法糖，有兴趣的话直接看[这个PR](https://github.com/rails/rails/pull/19157)。

```ruby
vehicles = ['Car', 'Bike', 'Truck', 'Bus']
vehicles.without("Car", "Bike") #=> ["Truck", "Bus"]
vehicles = {car: 'Hyundai', bike: 'Honda', bus: 'Mercedes', truck: 'Tata'}
vehicles.without(:bike, :bus) #=> {:car=>"Hyundai", :truck=>"Tata"}
```

### Array#second_to_last and Array#third_to_last

从后往前查找元素。

```ruby
['a', 'b', 'c', 'd', 'e'].second_to_last #=> "d"
['a', 'b', 'c', 'd', 'e'].third_to_last #=> "c"
```

### Integer#positive? and Integer#negative?

判断一个数是正还是负，该方法 Ruby 2.3 以及内建。

```ruby
4.positive? #=> true
4.negative? #=> false
-4.0.positive? #=> false
-4.0.negative? #=> true
```

### Array#inquiry

[PR](https://github.com/georgeclaghorn/rails/commit/c64b99ecc98341d504aced72448bee758f3cfdaf)，这是一个值得注意的特性。

利用 `ArrayInquirer` 将一个数组包裹起来，然后提供一些语法糖来查找其中的字符串内容。

利用 `Array#inquiry` 可以容易的创建 `ArrayInquirer` 对象。

```ruby
users = [:mark, :max, :david]

array_inquirer1 = ActiveSupport::ArrayInquirer.new(users)

# creates ArrayInquirer object which is same as array_inquirer1 above
array_inquirer2 = users.inquiry

array_inquirer2.class #=> ActiveSupport::ArrayInquirer

# provides methods like:

array_inquirer2.mark? #=> true

array_inquirer2.john? #=> false

array_inquirer2.any?(:john, :mark) #=> true

array_inquirer2.any?(:mark, :david) #=> true

array_inquirer2.any?(:john, :louis) #=> false
```


## 原文

* [Active Support Improvements in Rails 5](http://blog.bigbinary.com/2016/02/17/active-support-improvements-in-Rails-5.html)
