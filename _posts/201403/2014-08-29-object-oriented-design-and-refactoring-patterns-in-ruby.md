---
layout: post
title:  "Object-Oriented Design and Refactoring Patterns in Ruby"
date:   2014-08-29 14:31:00
categories: ruby
tags: refactoring
author: "Victor"
---

本文講述基於 Ruby 語言的面向對象設計，並介紹了一些可以用來重構代碼的設計模式。

## 简介：繼承，封裝，多態，鸭子类型

### Introduction

Improve the way you code. Increase readability and maintainability. 提高你的代码的方式。提高可读性和可维护性。

**Object Oriented principles 面向對象設計原則**

* Encapsulation 封装
* Inheritance 继承
* Polymorphism 多态性
* Duck typing (particular in Ruby) 鴨子類型

**Code smells**

* Duplicate Code 重複的代碼
* Long methods 過長的方法
* Data clumps
* Feature Envy
* Conditionals 條件
* Comments 註釋
* [More Code smells](/ruby/code-smells/)

**Simple examples**

[Before / After refactoring Code smell content](https://github.com/tutsplus/ruby-refactoring)

**Run the tests after refactoring**

Just run ```rake test```

### Encapsulation 封裝

封裝是實現面向對象程序設計的第一步。封裝就是把屬於同一類事物的共性（屬性和行爲）歸到一個類中。被封裝的對象通常被稱爲抽象數據類型。
只公開代碼的對外接口而隱藏具體實現。

下面的例子中，我們把 `Screencaster` 和 `Student` 的共同特徵封裝成了 `Person`。

```ruby
class Person
  def initialize first_name, last_name, gender
    @first_name = first_name
    @last_name = last_name
    @gender = gender
  end

  def full_name
    first_name + ' ' + last_name
  end

  private

  def first_name
    @first_name
  end

  def last_name
    @last_name
  end
end

class Screencaster < Person
  def initialize first_name, last_name, gender, tools
    super first_name, last_name, gender
    @tools = tools
  end
end

class Student < Person
  def initialize first_name, last_name, gender, perferred_language
    super first_name, last_name, gender
    @perferred_language = perferred_language
  end
end
```

### Inheritance 继承

在封裝的基礎上，继承主要實現重用代碼，節省開發時間。
下面的例子中，`Screencaster` 和 `Student` 繼承自 `Person` 各自有擴展。

```ruby
class Person
  attr_reader :first_name, :last_name

  def initialize first_name, last_name, gender
    @first_name = first_name
    @last_name = last_name
    @gender = gender
  end

  def full_name
    first_name + ' ' + last_name
  end
end

class Screencaster < Person
  def initialize first_name, last_name, gender, tools
    super first_name, last_name, gender
    @tools = tools
  end
end

class Student < Person
  def initialize first_name, last_name, gender, perferred_language
    super first_name, last_name, gender
    @perferred_language = perferred_language
  end
end
```

```bash
pry
require './inheritance'
Student.new('Bill', 'Gates', 'M', 'Visual Baisc')
```

### Polymorphism 多態性

```ruby
class Person
  attr_reader :first_name, :last_name, :gender

  def initialize first_name, last_name, gender
    @first_name = first_name
    @last_name = last_name
    @gender = gender
  end

  def full_name
    first_name + ' ' + last_name
  end

  def present
    raise NotImplementedError, 'Must be implemented by subtypes.'
    # %Q(Hello, my name is #{full_name}, My gender is #{gender}.)
  end
end

class Screencaster < Person
  def initialize first_name, last_name, gender, tools
    super first_name, last_name, gender
    @tools = tools
  end

  def present
    %Q(Welcome to Tuts+ Premium! My name is #{full_name} and I'm your tutor.)
  end
end

class Student < Person
  def initialize first_name, last_name, gender, perferred_language
    super first_name, last_name, gender
    @perferred_language = perferred_language
  end

  def present
    %Q(What's up, everyone? My name is #{full_name}
       and I'm happy to be in the community! I'm #{gender} by the way.)
  end
end
```

`NotImplementedError` 这个异常一般是需要你在派生中实现的接口。如果我们真的需要定义一个公共抽象类（或者抽象方法）来让子类来实现，又该如何做呢？

我们可以通过在调用方法时抛出 ```NotImplementedError``` 来防止方法被调用。

### 相關閱讀

* [ruby实现抽象类和抽象方法](http://www.blogjava.net/killme2008/archive/2007/02/06/98262.html)

### Duck typing

鸭子类型并不在乎其内在类型可能是什么。只要它像鸭子一样走路,像鸭子一样嘎嘎叫,那它就是只鸭子。对于面向对象设计的清晰性来说，鸭子类型至关重要。在面向对象设计思想中，有这样一个重要原则：对接口编码，不对实现编码。如果利用鸭子类型，实现这一原则只需极少的额外工作，轻轻松松就能完成。举个例子：对象若有 *push* 和 *pop* 方法，它就能当作栈来用；反之若没有，就不能当作栈。

```ruby
class Person
  attr_reader :first_name, :last_name, :gender

  def initialize first_name, last_name, gender
    @first_name = first_name
    @last_name = last_name
    @gender = gender
  end

  def talk
    "Hello!"
  end
end

class Animal
  def initialize name
    @name = name
  end

  def talk
    "Woof, meow, roar! One of these"
  end
end

class Bug
  def initialize type
    @type = type
  end

  def talk
    "Do bugs event talk..?!"
  end
end
```

通常来说，一旦你的代码中出现 `is_a?` 或者 `kind_of?` 的话，说明你的设计思路已经偏离了 Ruby 推荐的方式。这时候就该考虑使用鸭子对象的方式来重构你的代码，用 `respond_to?` 就对了。

### 相關閱讀

* [鸭子类型](http://zh.wikipedia.org/wiki/Duck_typing)
* [duck-typing-in-ruby](https://speakerdeck.com/zhanghandong/duck-typing-in-ruby)

## 重构可用的几种方法

### Extract Method 提炼函数

Extract Method （提炼函数）是最常用的重构手法之一。当看见一个过长的函数或者一段需要注释才能让人理解用途的代码，就应该将这段代码放进一个独立函数中。

简短而命名良好的函数的好处：首先，如果每个函数的粒度都很小，那么函数被复用的机会就更大；其次，这会使高层函数读起来就想一系列注释；再次，如果函数都是细粒度，那么函数的覆写也会更容易些。

```ruby
#BEFORE
class Post

  attr_reader :title, :date

  def initialize title, date
    @title = title
    @date = date
  end

  def body
    <<-RETURN
      RANDOM TEXT Ladyship it daughter securing procured or am moreover mr. Put
      sir she exercise vicinity cheerful wondered. Continual say suspicion
      provision you neglected sir curiosity unwilling.
    RETURN

  end

  def condensed_format
    return_string = ''
    return_string << "Title: #{title}"
    return_string << "Date: #{date.strftime "%Y/%m/%d"}"

    return_string
  end

  def full_format
    return_string = ''
    return_string << "Title: #{title}"
    return_string << "Date: #{date.strftime "%Y/%m/%d"}"
    return_string << "--\n#{body}"

    return_string
  end

end
```

```ruby
#AFTER
class Post

  attr_reader :title, :date

  def initialize title, date
    @title = title
    @date = date
  end

  def body
    <<-RETURN
      RANDOM TEXT Ladyship it daughter securing procured or am moreover mr. Put
      sir she exercise vicinity cheerful wondered. Continual say suspicion
      provision you neglected sir curiosity unwilling.
    RETURN

  end

  def condensed_format
    metadata
  end

  def full_format
    return_string  = metadata
    return_string << "--\n#{body}"

    return_string
  end

  private

  def metadata
    return_string = ''
    return_string << "Title: #{title}"
    return_string << "Date: #{date.strftime "%Y/%m/%d"}"

    return_string
  end

end

```

```ruby
#TEST
require 'minitest/spec'
require 'minitest/autorun'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe Post do
  before do
    @date = Time.new(2014,02,28)
    @post = Post.new("Fragmented Class", @date)
  end

  describe "when requested a condensed format" do

    it "shows the post's title" do
      @post.condensed_format.must_include "Fragmented Class"
    end

    it "shows the post's date" do
      @post.condensed_format.must_include "2014/02/28"
    end

  end

  describe "when requested a full format" do

    it "shows the post's title" do
      @post.full_format.must_include "Fragmented Class"
    end

    it "shows the post's date" do
      @post.full_format.must_include "2014/02/28"
    end

  end
end

```

### Extract Class 提炼类

某个类做了应该由2个类做的事。建立一个新类，将相关的字段和函数从旧类搬移到新类。

一个类应该是一个清楚地抽象，处理一些明确的责任。但是在实际工作中，类会不断成长扩展。你会在这儿加入一些功能，在哪加入一些数据。给某个类添加一项新责任时，你会觉得不值得为这项责任分离出一个单独的类。于是，随着责任不断增加，这个类会变得过分复杂。很快，你的类就会变成一团乱麻。


```ruby
#BEFORE
class Student
  attr_accessor :first_term_assiduity, :first_term_test, :first_term_behavior
  attr_accessor :second_term_assiduity, :second_term_test, :second_term_behavior
  attr_accessor :third_term_assiduity, :third_term_test, :third_term_behavior

  def set_all_grades_to grade
    %w(first second third).each do |which_term|
      %w(assiduity test behavior).each do |criteria|
        send "#{which_term}_term_#{criteria}=".to_sym, grade
      end
    end
  end

  def first_term_grade
    (first_term_assiduity + first_term_test + first_term_behavior) / 3
  end

  def second_term_grade
    (second_term_assiduity + second_term_test + second_term_behavior) / 3
  end

  def third_term_grade
    (third_term_assiduity + third_term_test + third_term_behavior) / 3
  end
end

```

```ruby
#AFTER
class Student

  attr_reader :terms

  def initialize
    @terms = [
      Term.new(:first),
      Term.new(:second),
      Term.new(:third)
    ]
  end

  def set_all_grades_to grade
    terms.each { |term| term.set_all_grades_to(grade) }
  end

  def first_term_grade
    term(:first).grade
  end

  def second_term_grade
    term(:second).grade
  end

  def third_term_grade
    term(:third).grade
  end

  def term reference
    terms.find { |term| term.name == reference }
  end
end

class Term

  attr_reader :name, :assiduity, :test, :behavior

  def initialize name
    @name      = name
    @assiduity = 0
    @test      = 0
    @behavior  = 0
  end

  def set_all_grades_to grade
    @assiduity = grade
    @test      = grade
    @behavior  = grade
  end

  def grade
    (assiduity + test + behavior) / 3
  end
end

```

```ruby
#TEST
require 'minitest/spec'
require 'minitest/autorun'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe Student do
  it "has a grade for all three terms" do
    student = Student.new
    student.set_all_grades_to 10

    student.first_term_grade.must_equal 10
    student.second_term_grade.must_equal 10
    student.third_term_grade.must_equal 10
  end
end

```

### Pull Up Method

Pull Up Method 是在子类中消除重复代码的极常见的一招。你要做的就是把重复的代码提取到基类。这会让你更合理的使用继承。

```ruby
#BEFORE
class Person
  attr_reader :first_name, :last_name

  def initialize first_name, last_name
    @first_name = first_name
    @last_name = last_name
  end

end

class MalePerson < Person
  def full_name
    first_name + " " + last_name
  end

  def gender
    "M"
  end
end

class FemalePerson < Person
  def full_name
    first_name + " " + last_name
  end

  def gender
    "F"
  end
end
```

```ruby
#AFTER
class Person
  attr_reader :first_name, :last_name

  def initialize first_name, last_name
    @first_name = first_name
    @last_name = last_name
  end

  def full_name
    first_name + " " + last_name
  end
end

class MalePerson < Person
  def gender
    "M"
  end
end

class FemalePerson < Person
  def gender
    "F"
  end
end
```

```ruby
#TEST
require 'minitest/spec'
require 'minitest/autorun'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe MalePerson do
  it "has a full name" do
    MalePerson.new("John", "Smith").full_name.must_equal "John Smith"
  end
end

describe MalePerson do
  it "has a full name" do
    FemalePerson.new("Michelle", "Smith").full_name.must_equal "Michelle Smith"
  end
end
```

### Rename Method

取一个好名字很重要。

```ruby
#BEFORE
class UserService
  USERNAME = "josemota"
  PASSWORD = "secret"

  class << self
    def login username, password
      username == USERNAME && password == PASSWORD
    end
  end
end
```

```ruby
#AFTER
class UserService
  USERNAME = "josemota"
  PASSWORD = "secret"

  class << self
    def sign_in username, password
      username == USERNAME && password == PASSWORD
    end
  end
end
```

```ruby
#TEST
require 'minitest/autorun'
require 'minitest/spec'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe UserService do
  it "can log in" do
    if ENV["BEFORE"]
      assert UserService.login("josemota", "secret")
    else
      assert UserService.sign_in("josemota", "secret")
    end
  end
end
```

### Move Field

Move Field 不是简单的把属性从一个类移动到另一个类。如果某个 field，在其所驻 class 之外的另一个 class 中有更多的函数使用了它，那么可以考虑将这个 field 移动到另一个 class。

```ruby
#BEFORE
PHONE_CODES = {
  en_gb: "44",
  pt:    "351"
}

class Phone
  attr_reader :number

  def initialize number
    @number = number
  end

  def to_s
    number
  end
end

class Person
  attr_reader :locale, :phone

  def initialize(locale: :en_gb, phone: nil)
    @locale = locale
    @phone = Phone.new phone
  end

  def full_phone
    ["+", PHONE_CODES[locale], " ", phone].join
  end
end
```

```ruby
#AFTER
PHONE_CODES = {
  en_gb: "44",
  pt:    "351"
}

class Phone
  attr_reader :number, :locale

  def initialize number, locale
    @number = number
    @locale = locale
  end

  def to_s
    PHONE_CODES[locale] + " " + number
  end
end

class Person
  attr_reader :phone

  def initialize(locale: :en_gb, phone: nil)
    @phone = Phone.new phone, locale
  end

  def full_phone
    ["+", phone].join
  end
end
```

```ruby
#TEST
require 'minitest/autorun'
require 'minitest/spec'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe Person do
  it "has a phone number" do
    person = Person.new(locale: :pt, phone: "555-0342")
    person.full_phone.must_equal "+351 555-0342"
  end
end
```

### Form Template Method 塑造模板方法

常见的一种情况是：你有一些子类，其中相应的某些函数以相同的顺序执行大致相近的操作，但是各操作不完全相同。这种情况下我们可以将执行的序列移至超类，并借助多态保证各子类中的操作仍得以保持差异性。这样的函数被称为 *Template Method（模板函数）*

**原则：继承是避免重复代码的一个强大工具。无论何时，只要你看见2个子类之中有类似的方法，就可以把它们提升到超类。但是如果这些函数并不完全相同该这么办？仍有必要尽量避免重复，但又必须保持这些函数之间的实质差异。**

```ruby
#BEFORE
class Ticket
  attr_reader :price

  def initialize
    @price = 2.0
  end

end

class SeniorTicket < Ticket
  def price
    @price * 0.75
  end
end

class JuniorTicket < Ticket
  def price
    @price * 0.5
  end
end
```

```ruby
#AFTER
class Ticket
  attr_reader :price

  def initialize
    @price = 2.0
  end

  def price *args
    @price * discount
  end

  def discount
    1
  end

end

class SeniorTicket < Ticket
  def discount
    0.75
  end
end

class JuniorTicket < Ticket
  def discount
    0.5
  end
end
```

```ruby
#TEST
require 'minitest/spec'
require 'minitest/autorun'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe Ticket do
  it "has a calculated price" do
    Ticket.new.price.must_equal 2.0
  end

  it "costs less if a senior" do
    SeniorTicket.new.price.must_equal 1.5
  end

  it "costs less if a junior" do
    JuniorTicket.new.price.must_equal 1.0
  end

end
```

### Parameterize Method 令函数携带参数

"令函数携带参数" 并不是简单的让你在函数里加上参数, 如果函数里需要某个参数, 我们谁都会加上它. 你可能发现这样的几个函数: 它们做着类似的事情, 只是因为极少的几个值导致函数的策略不同, 这时可以使用 *Parameterize Method* 消除函数中那些重复的代码了, 而且可以用这个参数处理其它更多变化的情况。

```ruby
#BEFORE
class Student
  def first_term_grade
    10
  end

  def second_term_grade
    11
  end

  def third_term_grade
    12
  end
end
```

```ruby
#AFTER
class Student
  GRADES = {
    first: 10,
    second: 11,
    third: 12
  }

  def term_grade index
    GRADES[index]
  end
end
```

```ruby
#TEST
require 'minitest/autorun'
require 'minitest/spec'

require 'before' if ENV["BEFORE"]
require 'after' unless ENV["BEFORE"]

describe Student do
  if ENV["BEFORE"]

    it "has a first grade" do
      Student.new.first_term_grade.must_equal 10
    end
    it "has a second grade" do
      Student.new.second_term_grade.must_equal 11
    end
    it "has a third grade" do
      Student.new.third_term_grade.must_equal 12
    end

  else # AFTER

    it "has a first grade" do
      Student.new.term_grade(:first).must_equal 10
    end
    it "has a second grade" do
      Student.new.term_grade(:second).must_equal 11
    end
    it "has a third grade" do
      Student.new.term_grade(:third).must_equal 12
    end

  end
end

```

### 视频链接

* [tutsplus.com](https://code.tutsplus.com/courses/object-oriented-design-and-refactoring-patterns-in-ruby)
