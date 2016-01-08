---
layout: post
title:  "Awesome Low Level Caching for Your Rails App"
date:   2016-01-08 03:00:00
categories: rails
tags: caching
author: "Victor"
---

## 高性能的缓存结构减少重复计算

Ruby 程序员都喜欢 DRY，那么如何设计缓存结构才能让我们的缓存代码不会被重复计算呢？

我们希望：

* 缓存结果不被改变前，绝不计算两次
* 数据库中的任何改变应该立刻呈现给用户，而不是等几分钟让缓存失效或过期
* 用户请求前不要预先计算缓存结果

### The best blog app ever

假设我们有一个 Blog 平台，用户可以在上面发表自己的帖子。每个帖子都可以被其他访客评论，每条评论都可以被顶和踩。

当一条评论被顶的次数多过被踩的次数，我们就认为它是 **精华评论**。`comment.score > 0`

如何计算一个用户发表的帖子产生了多少 **精华评论** 呢?

### Methods definitions without cache

首先让我们定义 `User`, `Post` 和 `Comment`:

```ruby
class Comment
  include Mongoid::Document
  include Mongoid::Timestamps

  field :content, type: String
  # stores the score for queries
  field :score, type: Integer

  belongs_to :post
  has_and_belongs_to_many :upvoters, class_name: 'User', inverse_of: 'liked_comments'
  has_and_belongs_to_many :downvoters, class_name: 'User', inverse_of: 'disliked_comments'

  before_save :set_score


  def set_score
    self.write_attributes(score: upvoters_ids.size - downvoters_ids.size)
  end
end
```

```ruby
class Post
  include Mongoid::Document
  include Mongoid::Timestamps

  field :content, type: String

  belongs_to :author, class_name: 'User', inverse_of: :post
  has_many :comments

  # How many interesting comments does it have?
  def interesting_comments_count
    comments.gt(score: 0).count #gt = greater than
  end
end
```

```ruby
class User
  include Mongoid::Document
  include Mongoid::Timestamps

  has_many :posts, class_name: 'Post', inverse_of: :author

  # the post score is the sum of all posts scores
  def interesting_comments_count
    posts.map(&:interesting_comments_count).reduce(:+) #map/reduce rules
  end
end
```

### The fastest blog app ever

为了让应用更快，我们需要添加一些缓存。

Rails 本身就有一个很棒的告诉缓存工具，可以在配置中指定 `Rails.cache` 使用 redis 来进行缓存。

一切设置好之后，我们可以使用 `Rails.cache.fetch( key, expires_in: seconds) do ...` 来触发缓存：

* 如果可以通过该 key 找到缓存结果则直接返回
* 如果没有找到结果或者 key 不存在，则执行代码块返回运算结果，并且把该结果赋值给 key 保存到缓存系统中

下面给 `comment.rb` 增加缓存：

```ruby
class Comment
  # this will update the `updated_at` key for our post
  belongs_to :post, touch: true
end
```

```ruby
#app/models/post.rb

def interesting_comments_count
  # when was the last update?
  date_key = self.updated_at

  # create unique key for each post, method, and timestamp
  cache_key = "postInterestingCommentCount#{id}|#{date_key}"

  # Fetch the value, or calculate it then store it into cache:
  Rails.cache.fetch(cache_key, expire_in: 2.days) do
    comments.gt(score: 0).count #gt = greater than
  end
end
```

说明：

1. 第一次运行，缓存 key 被创建，计算结果并赋值给该 key
2. 再次调用 `interesting_comments_count` 方法，如果评论分数没变，则不进行任何运算和数据库查询直接返回上次的结果
3. 如果有人更新了评论分数，帖子的时间戳就会变化。因此 `date_key` 变化也造成 `cache_key` 变化，回到步骤1.

可以看到，当缓存 key 过期之后我们并没有删除该缓存结果，只是忽略它重新创建一个新的缓存 key。

同样可以把这个技术代入到其他模型中：

```ruby
class Post
  belongs_to :author, class_name: 'User', inverse_of: :post, touch: true
end
```

```ruby
class User
  def interesting_comments_count
    cache_timestamp = self.updated_at
    cache_key = "userInterestingCommentCount#{id}|#{cache_timestamp}"

    Rails.cache.fetch(cache_key, expire_in: 2.days) do
      Post.map(&:interesting_comments_count).reduct(:+)
    end
  end
end
```

做完这一切之后让我们看看真实环境会发生什么。


1. 一个用户 `u` 有 10 个帖子
2. `u.interesting_comments_count` 被调用：
  * `u` 的每条 Post 生成一个缓存 key
  * `u` 也生成一个缓存 key
3. `u.interesting_comments_count` 再次被调用：
  * 直接触发缓存，不产生任何数据库查询
4. 当一条评论被顶过
  * 触发评论的时间戳更新
  * 评论所属的帖子，帖子所属的用户时间戳都更新
5. `u.interesting_comments_count` 被调用：
  * 用户的缓存 key 过期，所以该方法被再次执行
  * 该用户的 10 条帖子中有 9 条触发缓存结果，不会产生数据库查询
  * 刚刚被更新时间戳的帖子重新计算缓存结果，返回新的缓存 key
6. 回到步骤2

## 结论

大部分场景下，这一招都是很有用的。他可以让你随处调用该方法，而不用担心浪费大量时间去做重复计算。

看完这篇文章你肯定隐隐有一种想法，这特么就是 [fragment-caching](http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching) 搬家到 Model 层中了啊。没错！就是这么回事，但这样的好处在于不会出现缓存 key 是 `nil` 的情况，并且该方法你随时随地想用就用。

## 原文

* [Awesome Low Level Caching for Your Rails App](http://aurelien-herve.com/blog/2015/01/21/awesome-low-level-caching-for-your-rails-app/)
