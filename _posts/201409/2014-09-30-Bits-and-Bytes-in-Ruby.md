---
layout: post
title:  "Working with Bits and Bytes in Ruby"
date:   2014-09-30 11:50:00
categories: ruby
tags: tip
author: "Victor"
---

有时候我们会遇到如下需求：我们要读取一个 2进制 UDP 数据包，把其中的数据转换成 Ruby 可识别的字符串或数字什么的，然后发送一个响应什么的。

## Working with different bases

### Literals 字面值

Ruby 支持 **0b** 前缀的 2进制和 **0x** 前缀的 16进制。

```ruby
>> 0b1010
#=> 10
>> 0x4a2f81
#=> 4861825
```

### String#to_i(base = 10)

这是我们常用的方法，通常都是把一个字符串类型的数字转换成 Ruby 的数值，以便进行运算操作。

这个方法内部的实际意义是，把这个给定的字符串按照 10进制转换成一个数字。所以我们当然可以把字符串按照 2进制或者 16进制来转换。

```ruby
>> '0b1010'.to_i(2)
=> 10
>> '10111010'.to_i(2)
=> 186
>> '1af842'.to_i(16)
=> 1767490
```

### Fixnum#to_s(base = 10)

把 Fixnum 类型的对象，根据给定的进制类型转换成一个字符串。

```ruby
>> 24.to_s(16)
=> "18"
>> 0b101010.to_s(16)
=> "2a"
>> 42.to_s(2)
=> "101010"
>> rand(10**8..10**9).to_s(36)
=> "fd95me"
```

## Bitwise and Shift Operators 位移操作


Ruby 含有如下位移操作符：

* & bitwise AND
* | bitwise OR
* ^ bitwise XOR
* ~ bitwise NOT
* >> right shift
* << left shift

这部分可以阅读 [Ruby's bitwise operators](http://calleerlandsson.com/2014/02/06/rubys-bitwise-operators/)

### 把日期转换成 16-bit 的 integer 值

```
15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
 M  M  M  M  D  D  D  D  D  Y  Y  Y  Y  Y  Y  Y
```

这种设计有足够的长度来标示 12 月份 (4 bits = 16 values),  31 天 (5 bits = 32 values)  1900-2028 年 (7 bits = 128 values)。在真实的项目中，如果想要支持更大范围的年份，可以参考 Y2K 和 Y2038 标准。

April 6th, 2014 可以用如下方法转换：

```
0100 00110 1110010  (= 17266)
4    6     114
```

有兴趣可以想想怎么把 17266 反向解析成为一个可识别的日期。

## Packing and unpacking

关于 **String#unpack** 和 **Array#pack**，可以阅读 [Ruby pack unpack](http://blog.bigbinary.com/2011/07/20/ruby-pack-unpack.html) 和 [Les méthodes pack et unpack en Ruby](http://vfsvp.fr/article/les-methodes-pack-et-unpack-en-ruby/)


## 相关链接

* [Working with Bits and Bytes in Ruby](http://www.webascender.com/Blog/ID/529/Working-with-Bits-and-Bytes-in-Ruby#.VCoLPCmSz0Z)
* [Byte manipulation in Ruby](http://happybearsoftware.com/byte-manipulation-in-ruby.html)
* [8 bit binary conversion](https://www.ruby-forum.com/topic/4417836)
* [ULOG_ACCTD学习总结](http://blog.csdn.net/flyingstarwb/article/details/1874529)
