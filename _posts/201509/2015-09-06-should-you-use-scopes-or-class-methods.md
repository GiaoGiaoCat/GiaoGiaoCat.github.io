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

那么问题来了，既然可以使用类方法，为啥要使用 Scopes 来实现一样的功能呢？

### 当已经存在类方法的时候，何时使用 Scopes

例如：你希望根据特定过滤条件筛选出部分结果。当满足这些条件的结果集没有数据时，你希望返回全部数据。

如果使用 scope ：

```ruby
# app/models/review.rb

scope :created_since, ->(time) { where("reviews.created_at > ?", time) if time.present? }
```

如果使用类方法：

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

Scopes 帮我们做了一些额外的工作，因为它默认会返回 scope，所以我们可以使用方法链：

```ruby
Review.positive.created_since(5.days.ago)
```

但是使用类方法，则需要手动处理意外情况（你需要自己处理 **time** 是 `nil` 的情况）。

[方法应该返回同样的对象](http://www.justinweiss.com/blog/2014/06/24/simplify-your-ruby-code-with-the-robustness-principle/)，这样你就永远不用担心边界情况和意外错误。你可以假设你永远都会拿到你期待处理的对象。

回到我们的例子中，这意味着在使用方法链的时候，你不用担心其中某一个环节出现 `nil` 的情况。

但是仍有一些情况，让我们破坏 Scopes：

```ruby
# app/models/review.rb

scope :broken, -> { "Hello!!!" }
```

```
irb(main):001:0> Review.broken.most_recent(5)
NoMethodError: undefined method `most_recent' for "Hello!!!":String
```

这代码是临时工写的吧？

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
* [Search and Filter Rails Models Without Bloating Your Controller](http://www.justinweiss.com/blog/2014/02/17/search-and-filter-rails-models-without-bloating-your-controller/)
