---
layout: post
title:  "Rails 中如何获取当前星期数目"
date:   2016-01-11 13:30:00
categories: rails
tags: time
author: "Victor"
---

最近在工作中要实现一个功能：

1. 把某个日期换算成今年的第几周
2. 随意取出某一周的开始和结束时间

先说 **错误** 的做法，使用 `strftime`：

```
%U - Week number of the year. The week starts with Sunday. (00..53)
%W - Week number of the year. The week starts with Monday. (00..53)
```

### 为什么这是错误的做法？

`Date.strftime("%U|%W")` 返回的结果是以 0 为起始值。为了显示准确还需要在存储或展示的时候手动给值增加 1。

看起来不错，但是根据[这里的解释](http://www.timeanddate.com/calendar/days/)，可以知道计算方法并不是从周一或者周日来计算的。

如果今年的1月1号是周1-周4，那么今年就53周；如果1月1号是周5，6，日。那么今年就52周。

而 `strftime` 显然没办法处理这种情况，尤其是当你用这种方式想把第 53 周换算成日期的时候。

```ruby
Date.commercial(2015, 52, 1) #=> Mon, 21 Dec 2015
Date.commercial(2014, 53, 1) #=> ArgumentError: invalid date
```

### 正确的做法

使用 `Date.cweek`：

```ruby
Date.parse("2015-12-31").cweek #=> 53
Date.commercial(2015, 53, 1) #=> Mon, 28 Dec 2015
```


