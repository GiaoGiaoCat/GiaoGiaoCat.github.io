---
layout: post
title:  "Value Object"
date:   2015-09-26 21:30:00
categories: ruby
tags: refactoring
author: "Victor"
---

## 提取值对象 - Extracting Value Objects

通过值对象，可以让你的模型和控制器不那么臃肿。

### 什么是 Value Objects

> A small simple object, like money or a date range, whose equality isn’t based on identity.

* 值对象是依赖于它们的值而不是身份的一种简单对象。
* 它们通常是不可变的。例如：Ruby 标准库中的 Date, URI, Pathname 等。
* Rails 应用程序中定义的特定域的值对象也是如此（下面的代码中的`ATTRS`）。从 ActiveRecords 中提取它们是常见的重构方法。

```ruby
class Favorite < ActiveRecord::Base
  ATTRS = [:name, :age]
end
```

下面是一个正确的例子：

```ruby
class Money < Struct.new(:amount, :currency)
  def amount=(other_amount)
    Money.new(other_amount, currency)
  end
end

usd = Money.new(10, 'USD')
usd2 = Money.new(10, 'USD')

usd.hash == usd2.hash # => true
usd == usd2 # => true
```

下面就是一个错误的例子：

```ruby
Point = Value.new(:x, :y)
p = Point.new(1, 2)
p.x = 1
# => NoMethodError: undefined method x= for #<Point:0x00000100943788 @x=0, @y=1>
```

### 如何定义值对象

我们需要在值对象的类中实现 `==` 或 `<=>` 方法，之后就可以根据他们的值来比较两个值对象是否相等。

* 值对象应该包含多个属性
* 属性在其生命中期中应该是不可见的
* 两个值对象是否相等，取决于它们的 value (以及其 type、unit)

### 好处

主要是为了简化系统，重构代码而使用值对象。

1. 封装相关的逻辑放到一个类，遵从单一职责原则
2. 通过分割出与特定变量相关联的逻辑，减少大类的体积
3. 防止代码重复
4. 它允许我们将行为与数据结合起来，并在不污染模型的情况下为数据添加功能
5. 隔离之后，代码更容易测试


### 何时提取值对象

在 Rails 中，当你有一个属性或逻辑上互相关联的一小组属性，使用 Value Objects 特别方便。通常来说复杂的文本和数值都可被抽取成 Value Object。

例如：一个短消息系统，我需要处理电话号码的对象。在线商城系统需要一个 Money 类。Code Climate 有一个叫做 `Rating` 的 Value Object 用来表示 A - F 等级供给每个类和模块使用。最初我们使用 Ruby 的 String 实例来完成该功能，但是创建一个 Rating 的 Value Object 允许我把行为和数据结合起来，参考例1。

下面几种情况是提取值对象的明显特征：

1. 总是需要携带一堆参数 Arguments
2. 一个属性 attribute 跟着一个行为 behaviour
3. 两个不可分割的属性值 attributes value 和单位 unit，比如：金额
4. 可枚举的类

下面依次展示这 4 种情况。

### 1. Arguments together all the time

当你的代码中出现 “数据泥团” 的时候，就可以使用值对象了。比如我们很多方法都需要同时传递 `start_date` 和 `end_date`。

```ruby
class DateRange
  attr_reader :start_date, :end_date

  def initialize(start_date, end_date)
    @start_date, @end_date = start_date, end_date
  end

  def include_date?(date)
    date >= start_date && date <= end_date
  end

  def include_date_range?(date_range)
    start_date <= date_range.start_date && end_date >= date_range.end_date
  end

  def overlap_date_range?(date_range)
    start_date <= date_range.end_date && end_date >= date_range.start_date
  end

  def to_s
    "from #{start_date.strftime('%d-%B-%Y')} to #{end_date.strftime('%d-%B-%Y')}"
  end
end

class Event < ActiveRecord::Base
  def date_range
    DateRange.new(start_date, end_date)
  end

  def date_range=(date_range)
    self.start_date = date_range.start_date
    self.end_date = date_range.end_date
  end
end
```

```bash
$ event = Event.create(name: 'Ruby conf', start_date: Date.today, end_date: Date.today + 1.days)
$ event.date_range #=> #<DateRange:0x007fd8760c2690 @start_date=Tue, 06 Jun 2017, @end_date=Fri, 16 Jun 2017>
$ event.date_range.include_date?(Date.today) #=> true
$ event.date_range.include_date_range?(DateRange.new(Date.today, Date.today + 2.days)) #=> false
$ event.date_range.include_date_range?(DateRange.new(Date.today, Date.today + 1.days)) #=> true
```

另一个简单的例子。

```ruby
class Person < ActiveRecord::Base
  def address
    Address.new(address_city, address_state)
  end

  def address=(address)
    self.address_city = address.city
    self.address_state = address.state
  end
end

class Address
  attr_reader :city, :state

  def initialize(city, state)
    @city, @state = city, state
  end

  def ==(other_address)
    city == other_address.city && state == other_address.state
  end
end
```

```bash
$ gary = Person.create(name: "Gary")
$ gary.address_city = "Brooklyn"
$ gary.address_state = "NY"
$ gary.address #=> #<Address:0x007fcbfcce0188 @city="Brooklyn", @state="NY">

$ gary.address = Address.new("Brooklyn", "NY")
$ gary.address #=> #<Address:0x007fcbfa3b2e78 @city="Brooklyn", @state="NY">
```

#### 2. One attribute with behaviour

当你的模型中有一个属性，该属性需要关联一些行为，而这些行为跟模型毫无关系。假设你有一些房间，你需要按温度给这些房间排序。或者找出比较冷的房间。

我们当然可以认为 Room 应该知道自己冷不冷，但是还是建议抽出一个值对象，并把 `code?` 之类的方法放在 Temperature 中。这样的好处是，我将来也可以问椅子和书桌冷不冷。

```ruby
class Temperature
  include Comparable
  attr_reader :degrees
  COLD = 20
  HOT = 25

  def initialize(degrees)
    @degrees = degrees
  end

  def cold?
    self < COLD
  end

  def hot?
    self > HOT
  end

  def <=>(other)
    degrees <=> other.degrees
  end

  def hash
    degrees.hash
  end

  def to_s
    "#{degrees} °C"
  end
end
```

```bash
$ room_1 = Room.create(degrees: 10)
$ room_2 = Room.create(degrees: 20)
$ room_3 = Room.create(degrees: 30)
$ room_1.temperature.cold? #=> true
$ room_1.temperature.hot? #=> false
$ [room_1.temperature, Temperature.new(20), room_3.temperature, room_2.temperature].sort #=> [#<Temperature:0x007fe194378840 @degrees=10>, #<Temperature:0x007fe194378818 @degrees=20>, #<Temperature:0x007fe1943787c8 @degrees=20>, #<Temperature:0x007fe1943787f0 @degrees=30>]
$ [room_1.temperature, Temperature.new(20), room_3.temperature, room_2.temperature].uniq #=> [#<Temperature:0x007fe194361e88 @degrees=10>, #<Temperature:0x007fe194361e60 @degrees=20>, #<Temperature:0x007fe194361e38 @degrees=30>]
```

#### 3. Two inseparable attributes value and unit

```ruby
class Product < ActiveRecord::Base
  def cost
    Money.new(cents, currency)
  end

  def cost=(cost)
    self.cents = cost.cents
    self.currency = cost.currency.to_s
  end
end
```

```bash
$ product = Product.create(cost: Money.new(500, "EUR"))
$ product.cost #=> #<Money fractional:500 currency:EUR>
$ product.cost.cents #=> 500
$ product.currency #=> "EUR"
```

#### 4. Class enumerable

我们都写过下面这样的代码。

```ruby
class Event < ActiveRecord::Base
  SIZE = %w(
    small
    medium
    big
  )
end
```

这个属性和你模型的业务逻辑无关，而且当你有很多模型都有 size 属性的时候也很麻烦，所以可以用值对象。

```ruby
class Size
  SIZES = %w(small medium big)
  attr_reader :size

  def initialize(size)
    @size = size
  end

  def self.to_select
    SIZES.map{|c| [c.capitalize, c]}
  end

  def valid?
    SIZES.include?(size)
  end

  def to_s
    size.capitalize
  end
end
```

#### 重构

有一些方便我们使用的 gem，比如 [Values](https://github.com/tcrayford/Values) 和 [money](https://github.com/RubyMoney/money)。

我们是 Values gem 来重构一波。

```ruby
class DateRange < Value.new(:start_date, :end_date)
  def include_date?(date)
    date >= start_date && date <= end_date
  end

  def include_date_range?(date_range)
    start_date <= date_range.start_date && end_date >= date_range.end_date
  end

  def overlap_date_range?(date_range)
    start_date <= date_range.end_date && end_date >= date_range.start_date
  end

  def to_s
    "from #{start_date.strftime('%d-%B-%Y')} to #{end_date.strftime('%d-%B-%Y')}"
  end
end
```

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

Grade Score 是储存了某些值的东西。这些值可能不会保留在数据库中，我们只在业务逻辑中使用到它。值对象可能是：日期、金额、字符串。

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
* [ValueObject](https://martinfowler.com/bliki/ValueObject.html)
* [Value Objects Explained with Ruby](https://www.sitepoint.com/value-objects-explained-with-ruby/)
* [Good Value Object Conventions for Ruby](https://github.com/zverok/good-value-object)
* [Value Objects in Ruby on Rails](https://revs.runtime-revolution.com/value-objects-in-ruby-on-rails-9df64bc8db34)
