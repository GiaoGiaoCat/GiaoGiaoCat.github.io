---
layout: post
title:  "Ruby 元编程"
date:   2018-09-24 12:00:00

categories: ruby
tags: metaprogramming
author: "Victor"
---

作为一名以 Ruby 当作主语言的程序员，我一直认为元编程是编程中的一个边缘问题，对它的看法是 **元编程很厉害，但是咱们在业务开发中谁都别用它，不方便调试和理解。**

但是想一下，为什么在 libraries 和 gems 中处处充满了元编程，但是应用代码中几乎不存在呢？因为 gems 开发的目的是期望作出所有应用中可通用的组件。维基百科对 generalisation 的定义：通过抽象公共属性从特定实例中制定通用概念。

> One of the best ways to write flexible software is to write generic software. Instead of designing a single API that completely handles specific case, you write multiple APIs that handle smaller, more generic parts of that use case and then handling the entire case is just gluing those parts together.

> 编写灵活软件的最佳方法之一是编写通用软件。不是设计一个处理特定用例的 API，而是编写多个 API 来处理用例中更小、更通用的部分，最后将这些 API 组合在一起来处理这个用例。

看了下面的例子，在编写业务逻辑的时候适当使用元编程的技巧感觉也挺酷炫的。

### 例子1

```ruby
class Model
  def initialize; @attributes = {}; end

  def self.define_attribute(name)
    define_method name do
      @attributes.fetch(name)
    end

    define_method "#{name}=" do |value|
      @attributes[name] = value
    end
  end
end

class Talk < Model
  define_attribute :title
  define_attribute :abstract
end
```

```bash
talk = Talk.new
talk.title = "Metaprogramming for Generalists"
talk.title #=> "Metaprogramming for Generalists"
```

### 例子2

这个例子一步一步的展示了，如何开发一个简单的 dry-equalizer。

```ruby
class Address
  def ==(other)
    other.send(:street)   == send(:street) &&
    other.send(:city)     == send(:city) &&
    other.send(:country)  == send(:country)
  end
end

class GpsLocation
  def ==(other)
    other.send(:latitude)   == send(:latitude) &&
    other.send(:longitude)  == send(:longitude)
  end
end
```

```ruby
class Address
  def ==(other)
    [:street, :city, :country].all? { |key| other.send(key) == send(key) }
  end
end
```

```ruby
class Address
  def ==(other)
    keys = [:street, :city, :country]
    keys.all? { |key| other.send(key) == send(key) }
  end
end
```

```ruby
class Address
  define_method :== do |other|
    keys = [:street, :city, :country]
    keys.all? { |key| other.send(key) == send(key) }
  end
end
```

```ruby
class Address
  keys = [:street, :city, :country]
  define_method :== do |other|
    keys.all? { |key| other.send(key) == send(key) }
  end
end
```

```ruby
class Address
  keys = [:street, :city, :country]
  define_method :== do |other|
    keys.all? { |key| other.send(key) == send(key) }
  end
end
```

```ruby
class Address
  def self.equalize(*keys)
    define_method :== do |other|
      keys.all? { |key| other.send(key) == send(key) }
    end
  end

  equalize :street, :city, :country
end
```

```ruby
module Equalizer
  def equalize(*keys)
    define_method :== do |other|
      keys.all? { |key| other.send(key) == send(key) }
    end
  end
end

require "equalizer"

class Address
  extend Equalizer
  equalize :street, :city, :country
end
class GpsLocation
  extend Equalizer
  equalize :latitude, :longitude
end
```

```ruby
module Equalizer
  def equalize(*keys)
    define_method :== do |other|
      keys.all? { |key| other.send(key) == send(key) }
    end

    define_method :inspect do
      "#<#{self.class.name}#{keys.map { |key| " #{key}=#{send(key).inspect}" }.join}>"
    end
  end
end

class Address
  extend Equalizer
  equalize :street, :city, :country

  attr_reader :street, :city, :country

  def initialize(street, city, country)
    @street, @city, @country = street, city, country
  end
end

koko = Address.new("Chaoyang Lu 229", "Beijing", "China")
puts koko.inspect #=> #<Address street="Chaoyang Lu 229" city="Beijing" country="China">
```

### 相关链接

* [Rails… Still?!?!](https://blog.phusion.nl/2018/08/30/rails-still/)
* [dry-equalizer](https://github.com/dry-rb/dry-equalizer)
