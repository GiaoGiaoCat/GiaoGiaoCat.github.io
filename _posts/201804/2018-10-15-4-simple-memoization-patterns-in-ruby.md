---
layout: post
title:  "4 Simple Memoization Patterns in Ruby"
date:   2018-10-15 12:00:00

categories: ruby
tags: tip
author: "Victor"
---

Memoization 是一种可以用来加速访问者方法的技术。将耗时的方法逻辑只需要运行一次，然后将结果缓存。Rails 以前曾有一个类似的模块来处理这个类似情况 [memoize](https://apidock.com/rails/v3.2.13/ActiveSupport/Memoizable/memoize)，后来因为争议而被删除了，取而代之的下面讨论的一些常见的 memoization 模式。

### Super basic memoization

```ruby
class User < ActiveRecord::Base
  def twitter_followers
    # assuming twitter_user.followers makes a network call
    @twitter_followers ||= twitter_user.followers
  end
end
```

### Multi-line memoization

```ruby
class User < ActiveRecord::Base
  def main_address
    @main_address ||= begin
      maybe_main_address = home_address if prefers_home_address?
      maybe_main_address = work_address unless maybe_main_address
      maybe_main_address = addresses.first unless maybe_main_address
    end
  end
end
```

`begin...end` 会创建一个代码块，Ruby 会将其当作一个整体来对待，所以 `||=` 也会如期工作。

### What about nil?

这个模式有一个比较讨厌的问题，在第一个例子中，如果用户没有 twitter 账户或者 twitter API 返回 nil 咋办？在第二个例子中，如果没有任何地址造成整个代码块返回 nil 咋办？

因为每次调用该方法时，变量均为 nil，所以后面的代码逻辑都会执行一次。这时候 `||=` 就不是正确的选择了，我们需要区分 `nil` 和 `undefined`。

```ruby
class User < ActiveRecord::Base
  def twitter_followers
    return @twitter_followers if defined? @twitter_followers
    @twitter_followers = twitter_user.followers
  end
end
```

```ruby
class User < ActiveRecord::Base
  def main_address
    return @main_address if defined? @main_address
    @main_address = begin
      main_address = home_address if prefers_home_address?
      main_address ||= work_address
      main_address ||= addresses.first # some semi-sensible default
    end
  end
end
```

虽然这样的代码很难看，但是在 `nil`, `false` 和其它情况下都能正常工作。

### And what about parameters?

现在访问器方法都能正常工作了，如何用这个技巧来处理带参数的方法呢？

```ruby
class City < ActiveRecord::Base
  def self.top_cities(order_by)
    where(top_city: true).order(order_by).to_a
  end
end
```

我们可以用 Ruby 的 Hash 序列化技巧来处理这个问题。

```ruby
Hash.new {|h, key| h[key] = some_calculated_value }
```

```ruby
class City < ActiveRecord::Base
  def self.top_cities(order_by)
    @top_cities ||= Hash.new do |h, key|
      h[key] = where(top_city: true).order(key).to_a
    end
    @top_cities[order_by]
  end
end
```

不管在 order_by 中传递什么参数过来，都可以被缓存在 Hash 中。幸运的是，即便你传的参数是数组也没问题。

```ruby
h = {}
h[["a", "b"]] = "c"
h[["a", "b"]] # => "c"
```

### Why go through all this trouble?

但是，一旦你将这个技巧引入你的项目，并且咣咣咣的使用，那么代码很快会变得不可读。建议使用 [memoist](https://github.com/matthewrudy/memoist) 来搞定。

## 原文链接

* [4 Simple Memoization Patterns in Ruby](https://www.justinweiss.com/articles/4-simple-memoization-patterns-in-ruby-and-one-gem/)
