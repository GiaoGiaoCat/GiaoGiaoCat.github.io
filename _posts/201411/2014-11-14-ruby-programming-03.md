---
layout: post
title:  "Ruby 基础教程第4版读书笔记 3"
date:   2014-11-14 15:00:00
categories: ruby
tags: learningnote
author: "Victor"
---

## Ruby 的类

### 数值类

* Numeric 数值
  * Integer 整数
    * Fixnum 普通的整数
    * Bignum 大整数
  * Float 浮点小数
  * Rational 有理数
  * Complex 复数

Fixnum 表示计算机硬件可以处理的数值，Bignum 表示比 Fixnum 更大的数值。Ruby 会自动处理，不用关注这一点。

Rational 对象用 Rational(分子，分母) 的形式定义。

```ruby
a = Rational(2, 5)
puts a #=> (2/5)
```

#### 数值的字面量

**0b** 开头表示 2进制，**0** 或者 **0o** 开头表示 8进制， **0d** 开头表示 10进制，**0x** 开头表示 16进制。

字面量 | 作用(括号内为 10 进制的值)
--- | ---
123 | 表示10进制123
0d123 | 表示8进制整数(123)
0123 | 表示8进制整数(83)
0o123 | 表示8进制整数(83)
0x123 | 表示16进制整数(291)
0b1111011 | 表示2进制整数(123)
123.45 | 浮点小数
1.23e4 | 浮点小数指数表示法
1.23E-4 | 浮点小数指数表示法

字面量中的 _ 会被自动忽略。

```ruby
puts 1_123 #=> 1123
puts ob1111 #=>
```

#### 相关阅读

* [Working with Bits and Bytes in Ruby](/ruby/Bits-and-Bytes-in-Ruby/)

#### 算数运算

负整数的乘方返回的结果是表示有理数的 Rational 对象。

* ```x.div(y)``` 返回 x 除以 y 以后的商的整数
* ```x.quo(y)``` 返回 x 除以 y 后的商，如果 x，y 都是整数则返回 Rational 对象
* ```x.modulo(y)``` 求余
* ```x.divmod(y)``` 将 x 除以 y 后的商和余数作为数组返回
* ```x.remainder(y)``` 返回 x 除以 y 的余数

#### 除数为 0 时

除数为 0 时，Integer 类会返回错误，而 Float 类则返回 Infinity（无限大）或 NaN（Not a Number）。如果用这两个值进行运算，结果值返回 Infinity 或 NaN。应当注意避免这样的事情发生。

#### 近似值误差

Float 类的浮点小数用 2 取幂后的倒数表示，如 1/2, 1/4, 1/8 等。因此，处理 1/3 这样的 2进制无法正确表示的数值时，结果会产生误差。而如果要用 2进制的和来表示这类数值的话，就产生了近似值误差。

如果可以把小数转换为两个整数相除的形式，那么通过使用 Rational 类进行运算，可以避免近似值误差。

```ruby
a = Rational(1, 10) + Rational(2, 10)
b = Rational(3, 10)
p a == b #=> 10
```

```ruby
def rount_to_money(n)
  (n * 100).round / 100.0
end
```

#### 相关阅读

* [十进制小数转化为二进制小数](http://www.cnblogs.com/xkfz007/articles/2590472.html)

#### 数值类型的转换

* round 方法四舍五入
* ceil 返回比接收者大的最小整数
* floor 返回比接收者小的最大整数
* to_r 和 to_c 转换成 Rational 和 Complex 对象

#### 随机数

* ```Random.rand``` 方法返回比 1 小的浮点数
* ```Random.rand(参数)``` 参数为正整数时，返回 0 到该正数之间的数值
* 随机种子一样，可能出现重复值
* ```Random.new``` 不指定参数，生成随机种子数，在种子上调用 ```rand``` 方法，生成的随机数更好

#### 其它

* Comparable 模块封装了比较运算符
* Math 模块提供三角函数，对数函数等常用的函数运算方法

