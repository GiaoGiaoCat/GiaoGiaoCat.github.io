---
layout: post
title:  "提取值对象"
date:   2015-09-26 21:30:00
categories: ruby
tags: refactoring
author: "Victor"
---

## 提取值对象 - Extracting Value Objects

值对象是依赖于它们的值而不是身份的一种简单对象。它们通常是不可变的。例如：Ruby 标准库中的 Date, URI, Pathname 等，Rails 应用程序中定义的特定域的值对象也是如此（下面的代码中的`ATTRS`）。从 ActiveRecords 中提取它们是常见的重构方法。

```ruby
class Favorite < ActiveRecord::Base
  ATTRS = [:name, :age]
end
```

在 Rails 中，当你有一个属性或逻辑上互相关联的一小组属性，使用 Value Objects 特别方便。通常来说复杂的文本和数值都可被抽取成 Value Object。

例如：一个短消息系统，我需要处理电话号码的对象。在线商城系统需要一个 Money 类。Code Climate 有一个叫做 `Rating` 的 Value Object 用来表示 A - F 等级供给每个类和模块使用。最初我们使用 Ruby 的 String 实例来完成该功能，但是创建一个 Rating 的 Value Object 允许我把行为和数据结合起来，参考例1。

## 好处

* 防止代码重复
* 通过分割出与特定变量相关联的逻辑，减少大类的体积
* 封装相关的逻辑放到一个类，遵从单一职责原则

## 例子

### 例1

```ruby
class Rating
  include Comparable

  def self.from_cost(cost)
    if cost <= 2
      new("A")
    elsif cost <= 4
      new("B")
    elsif cost <= 8
      new("C")
    elsif cost <= 16
      new("D")
    else
      new("F")
    end
  end

  def initialize(letter)
    @letter = letter
  end

  def better_than?(other)
    self > other
  end

  def <=>(other)
    other.to_s <=> to_s
  end

  def hash
    @letter.hash
  end

  def eql?(other)
    to_s == other.to_s
  end

  def to_s
    @letter.to_s
  end
end
```

```ruby
class ConstantSnapshot < ActiveRecord::Base
  # …

  def rating
    @rating ||= Rating.from_cost(cost)
  end
end
```

### 例2

```ruby
# app/lib/report_card.rb

class ReportCard
  attr_accessor :grades

  def initialize(attributes = {})
    @scores = attributes[:scores]
    @grades ||= grade_scores
  end

  private

  def grade_scores
    @scores.map do |score|
      grade_score(score)
    end
  end

  def grade_score(score)
    if score < 60
      'F'
    elsif score < 70
      'D'
    elsif score < 80
      'C'
    elsif score < 90
      'B'
    else
      'A'
    end
  end
end
```

`ReportCard` 的 `grade_scores` 这部分很逻辑很多，我们可以把这些逻辑抽取到一个单独的 `Grade` 对象：

```ruby
# app/lib/grade.rb

class Grade
  attr_reader :score

  def initialize(score)
    @score = score
  end

  def letter
    grade_score(score)
  end

  private

  def grade_score(score)
    if score < 60
      'F'
    elsif score < 70
      'D'
    elsif score < 80
      'C'
    elsif score < 90
      'B'
    else
      'A'
    end
  end
end
```

```ruby
# app/lib/report_card.rb

class ReportCard
  attr_accessor :grades

  def initialize(attributes = {})
    @scores = attributes[:scores]
    @grades ||= grade_scores
  end

  private

  def grade_scores
    @scores.map do |score|
      Grade.new(score).letter
    end
  end
end
```

## 相关文章

* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Extracting Value Objects](http://gespinosa.org/2015/extracting-value-objects/)
