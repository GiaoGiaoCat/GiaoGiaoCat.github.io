---
layout: post
title:  "如何在 Ruby 中进行版本号的比较"
date:   2018-11-18 12:00:00

categories: ruby
tags: tip
author: "Victor"
---

在进行 API 开发的时候，经常会遇到客户端传递版本号，然后服务端进行不同的逻辑处理。用字符串比较经常遇到问题

```shell
'1.8.9' > '1.78.1' #=> true
```

## 方案 1

先介绍最优方案，使用 Ruby 的标准库中的 Gem::Version。不需要引用，直接用就行。

```shell
# Compares if a version is greater than other.
Gem::Version.new('2.1.15') > Gem::Version.new('1.14.1')
# => true

# It supports any number of minor versions
Gem::Version.new('2.0.0.1') < Gem::Version.new('2.0.1')
# => true

# And it deals with empty strings and nil values
Gem::Version.new('') < Gem::Version.new('2.0.1')
# => true
Gem::Version.new(nil) < Gem::Version.new('2.0.1')
# => true
```

## 方案 2

自己撸个类。

```ruby
class Version < Array
  def initialize s
    super(s.split('.').map { |e| e.to_i })
  end
  def < x
    (self <=> x) < 0
  end
  def > x
    (self <=> x) > 0
  end
  def == x
    (self <=> x) == 0
  end
end
```

```ruby
p [Version.new('1.2') < Version.new('1.2.1')]
p [Version.new('1.2') < Version.new('1.10.1')]
```

## 方案 3

使用别人写的 gem，[versionomy](https://github.com/dazuma/versionomy)

```ruby
require 'versionomy'

v1 = Versionomy.parse('0.1')
v2 = Versionomy.parse('0.2.1')
v3 = Versionomy.parse('0.44')

v1 < v2  # => true
v2 < v3  # => true

v1 > v2  # => false
v2 > v3  # => false
```
