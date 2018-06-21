---
layout: post
title:  "Ruby 基础教程第4版读书笔记 8 - Time 类与 Date 类"
date:   2014-12-09 12:20:00
categories: ruby
tags: learningnote time
author: "Victor"
---

时间除了表示年月日时分秒的信息以外，还包含了表示地域时差的时区(time zone)信息。

## 时间和日期的获取

* ```Time.mktime``` 可以根据制定时间获取 Time 对象
* 使用 Date 类需要引用 date 库。

```ruby
p t = Time.now #=> 2014-12-09 10:42:17 +0800
p t.year #=> 2014
```

```ruby
require 'date'
Date.today #=> #<Date: 2014-12-09 ((2457001j,0s,0n),+0s,2299161j)>
puts Date.today #=> 2014-12-09
```

## 时间和日期的计算

* Time 对象之间可以比较，运算，还可以增加或减少 Time 对象的秒数。

```ruby
t1 = Time.now
sleep(10)
t2 = Time.now
p t1 < t2 #=> true
p t2 - t1 #=> 11.201311

#增加 24小时的时间
t3 = t2 + 60 * 60 * 24 #=> 2014-12-10 11:13:02 +0800
```

* Date 对象之间的运算以天为单位。
* 日期减法运算的结果不是整数，而是 Rational 对象。
* Date 的加法，减法等运算，返回该对象前后的日期。
* ```>>, <<``` 对象可以返回上一个月或下一个月的相同日期的 Date 对象，如果没有相同日期则返回月末的日期。

```ruby
require 'date'
d1 = Date.new(2013, 1, 1)
d2 = Date.new(2013, 1, 4)
puts d2 - d1 #=> (3/1) # 3天的意思

puts d2 + 2 #=> 2013-01-06
puts d2 << 1 #=> 2012-12-04
```

## 时间和日期的格式

* ```t.strftime(format)```
* ```t.to_s```
* ```t.tfc2822``` 生成符合电子邮件头部信息中的 Date
* ```t.iso8601``` 生成符合 ISO 8601 国际标准的时间格式的字符串
* ```t.utf``` 把 Time 对象的时区变为国际协调时间
* ```t.localtime``` 把 Time 对象从 UTC 时间变更为本地时间

```ruby
t = Time.now #=> 2014-12-09 11:27:29 +0800
t.utc #=> 2014-12-09 03:27:29 UTC
t.localtime #=> 2014-12-09 11:27:29 +0800
```

## 从字符串中获取时间

要使用 ```parse``` 方法必须先引入 time 或者 date 库。

```ruby
require 'time'

p Time.parse("Sat Mar 30 03:54:15 UTC 2013") #=> 2013-03-30 03:54:15 UTC
p Time.parse("Sat, 30 Mar 2013 03:54:15 +0800")
p Time.parse("2013/10/13")
p Time.parse("2013/03/01 03:54:15")
p Time.parse("H25.03.31") #=> 2013-03-31 00:00:00 +0800
p Time.parse("S48.8.24") #=> 1973-08-24 00:00:00 +0800
```

```ruby
require 'date'
puts Date.parse("H26.03.30") #=> 2014-03-30
```


## 相关

* [strftime 可以使用的参数说明](/rails/rails-time/)
