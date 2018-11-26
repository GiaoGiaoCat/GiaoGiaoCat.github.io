---
layout: post
title:  "How to Check If a Variable is Defined in Ruby"
date:   2018-11-25 12:00:00

categories: ruby
tags: tip
author: "Victor"
---

## `defined?` 关键字

Ruby 的 `defined?` 关键字可以用来检查一个变量是否被定义了。

* 如果已经定义则返回变量类型
* 没定义就返回 `nil`

```ruby
apple = 1
defined?(apple)
# "local-variable"

defined?(bacon)
# nil
```

它很像 JavaScript 中的 `typeof` 操作符，如果想拿到变量是哪个类的对象可以使用 `class` 方法。

下面有一些你需要注意的点：

* `defined?` 是 关键字 而不是 方法
* `defined?` 是 Ruby 中少数以 `?` 结尾的东西之一，但是没有按照惯例返回布尔值
* `defined?` 可以识别变量的值是 `nil` 和从未设置过的变量之间的区别

## 检查方法是否被定义

因为 `defined?` 是关键字而非方法，所以你不可以在某对象上调用 `defined?`，这时候我们需要的是 `respond_to?`。

```ruby
[].respond_to?(:size)
# true
[].respond_to?(:orange)
# false
```

## 检查常量是否存在

检查一个 `mod` 及其祖先中是否定义某常量。

* `mod` 可以是 class 或者 module
* 如果 `mod` 是 module 的话，会附加检查 `Object` 及其祖先

```ruby
Object.const_defined?(:String)
# true
Object.const_defined?(:A)
# false
```

## 原文链接

* [How to Check If a Variable is Defined in Ruby](https://www.rubyguides.com/2018/10/defined-keyword/)
