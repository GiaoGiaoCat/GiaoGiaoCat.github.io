---
layout: post
title:  "Should You Use Scopes or Class Methods?"
date:   2015-09-06 17:00:00
categories: rails
tags: tip
author: "Victor"
---

Scopes 可以帮我们从数据库中获取正确的数据：

```ruby
# app/models/review.rb
class Review < ActiveRecord::Base
  scope :most_recent, -> (limit) { order("created_at desc").limit(limit) }
end

# app/models/homepage_controller.rb
@recent_reviews = Review.most_recent(5)
```

当然，我们也可以用类方法来达到同样的目的：

```ruby
# app/models/review.rb
def self.most_recent(limit)
  order("created_at desc").limit(limit)
end

# app/models/homepage_controller.rb
@recent_reviews = Review.most_recent(5)
```

### Scopes 其实就是类方法

在 Rails 内部 Active Record 会把 Scopes 转换成类方法。

```ruby
# File activerecord/lib/active_record/scoping/named.rb, line 141
def scope(name, body, &block)
  unless body.respond_to?(:call)
    raise ArgumentError, 'The scope body needs to be callable.'
  end

  if dangerous_class_method?(name)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" "                "on the model \"#{self.name}\", but Active Record already defined "                "a class method with the same name."
  end

  extension = Module.new(&block) if block

  singleton_class.send(:define_method, name) do |*args|
    scope = all.scoping { body.call(*args) }
    scope = scope.extending(extension) if extension

    scope || all
  end
end
```

那么问题来了，既然可以使用类方法，为啥要使用 Scopes 来实现一样的功能呢？

### 当已经存在类方法的时候，何时使用 Scopes

例如：你希望根据特定过滤条件筛选出部分结果。当满足这些条件的结果集没有数据时，你希望返回全部数据。

如果使用 scope ：

```ruby
# app/models/review.rb

scope :created_since, ->(time) { where("reviews.created_at > ?", time) if time.present? }
scope :recent, -> { order("reviews.updated_at DESC") }
```

```
Review.created_since(nil).recent
# SELECT "reviews".* FROM "reviews" ORDER BY reviews.created_at DESC

Review.created_since('').recent
# SELECT "reviews".* FROM "reviews" ORDER BY reviews.created_at DESC
```

使用类方法:

```ruby
def self.created_since(time)
  where("reviews.created_at > ?", time) if time.present?
end

def self.recent
  order("reviews.updated_at DESC")
end
```

```
Review.created_since('').recent
NoMethodError: undefined method `recent' for nil:NilClass
```

Scopes 帮我们做了一些额外的工作，因为它默认会返回 scope，所以我们可以使用方法链：

```ruby
Review.positive.created_since(5.days.ago)
```

但是使用类方法，则需要手动处理意外情况（你需要自己处理 **time** 是 `nil` 的情况）。

```ruby
# app/models/review.rb

def self.created_since(time)
  if time.present?
    where("reviews.created_at > ?", time)
  else
    all
  end
end
```

[方法应该返回同样的对象](http://www.justinweiss.com/blog/2014/06/24/simplify-your-ruby-code-with-the-robustness-principle/)，这样你就永远不用担心边界情况和意外错误。你可以假设你永远都会拿到你期待处理的对象。

回到我们的例子中，这意味着在使用方法链的时候，你不用担心其中某一个环节出现 `nil` 的情况。

但是仍有一些临时工写的代码，让我们破坏 Scopes：

```ruby
# app/models/review.rb

scope :broken, -> { "Hello!!!" }
```

```
irb(main):001:0> Review.broken.most_recent(5)
NoMethodError: undefined method `most_recent' for "Hello!!!":String
```

### Scopes 是可以扩展的

以常见的分页 gem `kaminari` 为例：

```ruby
Post.page(2).per(15)

posts = Post.page(2)
posts.total_pages # => 2
posts.first_page? # => false
posts.last_page?  # => true
```

我们可以给 Scopes 增加扩展功能，这些功能只有在该 Scope 被调用之后才能使用。

```ruby
scope :page, -> num { # some limit + offset logic here for pagination } do
  def per(num)
    # more logic here
  end

  def total_pages
    # some more here
  end

  def first_page?
    # and a bit more
  end

  def last_page?
    # and so on
  end
end
```

在 Scopes 内部提供扩展方法是一种灵活强大的技术，当然我们也可以使用类方法来实现一样的功能。

```ruby
def self.page(num)
  scope = # some limit + offset logic here for pagination
  scope.extend PaginationExtensions
  scope
end

module PaginationExtensions
  def per(num)
    # more logic here
  end

  def total_pages
    # some more here
  end

  def first_page?
    # and a bit more
  end

  def last_page?
    # and so on
  end
end
```

### 什么时候使用类方法，而不是 Scopes

Scopes 通常被用来实现一些简单的过滤，然后用方法链关联在一起取出正确的对象集合。

下面的两种情况，不建议使用 Scopes：

  * 需要预先加载的时候，[I turn them into associations instead.](http://www.justinweiss.com/blog/2015/06/23/how-to-preload-rails-scopes/)
  * 需要使用多个内置的 scopes 方法（`where, limit`）时，使用类方法更好

换句话说当你的 Scopes 变得复杂时候，就是时候把它变成类方法了。

在类方法里，可以混合使用 Ruby 代码和数据库方法。比如，你可以在先从数据库中取出结果集，然后通过 Ruby 的 *sort_by* 方法来排序。

你也可以在类方法里面，从不同的地方取出结果。比如 Redis，Database，第三方API。然后把结果集合并成为一个 Ruby 的集合。

当然，你可以在 Scopes 里面使用 `selecting, sorting, joining, and filtering` 的代码，然后在类方法中使用这些 scopes。最后你的类方法会变得很清晰，并且 scopes 可以在其他地方复用。


## 相关连接

* [Should You Use Scopes or Class Methods?](http://www.justinweiss.com/blog/2015/09/01/should-you-use-scopes-or-class-methods/)
* [Active Record scopes vs class methods](http://blog.plataformatec.com.br/2013/02/active-record-scopes-vs-class-methods/)
* [Search and Filter Rails Models Without Bloating Your Controller](http://www.justinweiss.com/blog/2014/02/17/search-and-filter-rails-models-without-bloating-your-controller/)
