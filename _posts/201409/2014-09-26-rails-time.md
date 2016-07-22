---
layout: post
title:  "Rails 中的时间处理"
date:   2014-09-26 14:10:00
categories: rails
tags: time
author: "Victor"
---

### Time、Date、Datetime 的转换和区别

* Time 主要是对底层的C库时间函数的封装。这些函数通常基于UNIX纪元，因此不能表示1970年前的时间。
* Date 是为了弥补 Time 的缺点创建的，它能轻松地处理较早的时间，如达芬奇的生日（1452年4月15日），还能够智能处理历法改革的日期。但它也有缺点，如不能处理达芬奇的出生时间，它只能处理日期。
* DateTime 继承了 Date，旨在比上述两者都好。它可以像 Date 那样表示日期，也可以像 Time 那样表示时间。它通常是表示日期——时间值的推荐方式。

###格式化输出

```ruby
DateTime.parse(Time.now.to_s).strftime('%Y-%m-%d %H:%M:%S').to_s#就是按照2009-5-14 8：42：13的给定格式输出
```

###反向格式化

```ruby
DateTime.parse(params['start_date']).strftime('%Y-%m-%d %H:%M:%S').to_s
```

###集成多种方式输出

```ruby
# config/initializers/date_time_formats.rb
Time::DATE_FORMATS.merge!(
  :full => '%B %d, %Y at %I:%M %p',
  :md => '%m/%d',
  :mdy => '%m/%d/%y',
  :time => '%I:%M %p'
)
```

你就可以简单的通过调用

```ruby
Time.now.to_s(:full)#按照之前定义"May 14, 2009 at 08:39 AM"
```

###满足变化需求的输出。比如，要求是当前年份，不显示年，其他的年才显示

```ruby
Time::DATE_FORMATS.merge!(
  :friendly => lambda { |time|
    if time.year == Time.now.year
      time.strftime "%b #{time.day.ordinalize}"
    else
      time.strftime "%b #{time.day.ordinalize}, %Y"
    end
  }
)
```

```bash
>> Time.now.to_s(:friendly)
=> "May 14th"
>> (Time.now-2.years).to_s(:friendly)
=> "May 14th, 2007"
```

###to_s(format = :default) public

*:db* format outputs time in *UTC*; all others output time in *local*. Uses TimeWithZone’s *strftime*, so %Z and %z work correctly.

```ruby
:db           # => 2008-12-25 14:35:05
:number       # => 20081225143505
:time         # => 14:35
:short        # => 25 Dec 14:35
:long         # => December 25, 2008 14:35
:long_ordinal # => December 25th, 2008 14:35
:rfc822       # => Thu, 25 Dec 2008 14:35:05 +0000
```

###可以使用的参数说明

This is a major reminder, as I constantly have to search for this one whenever it comes to formatting dates and times in Ruby / Rails:

```ruby
t = Time.now                        #=> 2007-11-19 08:37:48 -0600
t.strftime("Printed on %m/%d/%Y")   #=> "Printed on 11/19/2007"
t.strftime("at %I:%M%p")            #=> "at 08:37AM
```

```
Year
%Y     year with century 2011
%y     year without century 11
%C     century number (year divided by 100) 20

Month
%B     full month name January
%b     abbreviated month name Jan
%h     same as %b Jan
%m     month as number (01-12)

Week
%U     week number of the year, Sunday as first day of week (00-53)
%W     week number of the year, Monday as first day of week (00-53)

Day
%A     full weekday name Wednesday
%a     abbreviated weekday name Wed
%d     day of the month (01-31)
%e     day of the month, single digits preceded by space ( 1-31)
%j     day of the year (001-366)
%w     weekday as a number, with 0 representing Sunday (0-6)
%u     weekday as a number, with 1 representing Monday (1-7)

Time
%H    hour (24-hour clock) (00-23)
%k     hour (24-hour clock); single digits preceded by space ( 0-23)
%I     hour (12-hour clock) (01-12)
%l     hour (12-hour clock); single digits preceded by space ( 1-12)
%M     minute (00-59)
%S     seconds (00-59)
%p     either AM or PM AM
%Z     timezone name or abbreviation EDT
%z     timezone offset from UTC -0400

Summaries
%D     date, same as %m/%d/%y 05/16/07
%v     date, same as %e-%b-%Y 16-May-2007
%F     date, same as %Y-%m-%d 2007-05-16
%R     time, 24 hour notation, same as %H:%M 18:06
%T     time, 24 hour notation, same as %H:%M:%S 18:06:15
%r     time, am/pm notation, same as %I:%M:%S %p 06:06:15 PM

Formatting
%n     newline character
%t     tab character
%%     percent character

Less common formats
%s    number of seconds since the Epoch, UTC
%c     national date and time representation
%+    national date and time representation
%x     national date representation
%X     national time representation
%G     year with century, starting on first Monday where week has 4 or more days.
%g    year without century, starting on first Monday where week has 4 or more days.
%V     week number of the year, starting on first Monday where week has 4 or more days.
```

## Rails 4 之后推荐的另外一种做法

Rails 4 推荐在 I18N 文件中指定时间格式，比如 `config/locales/en.yml`

```
en:
  time:
    formats:
      submitted: "%b %d %Y"
```

然后在 view 中使用 Rails 的 `#l` 辅助方法，`<%= l review.created_at, format: :submitted %>`。
详情可以参阅文档，http://guides.rubyonrails.org/i18n.html#adding-date-time-formats

### 相关链接

* [Date and Time in Ruby](http://www.tutorialspoint.com/ruby/ruby_date_time.htm)
