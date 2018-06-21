---
layout: post
title:  "发布 / 订阅模式"
date:   2015-03-30 12:20:00
categories: ruby
tags: refactoring
author: "Victor"
---

## 使用场景

很多项目中都有消息分发或者事件通知机制，尤其是模块化程度高的项目。

比如：在你的系统中，很多模块都对 **新建用户** 感兴趣。权限模块希望给新用户设置默认权限，报表模块希望重新生成当月的报表，邮件系统希望给用户发送激活邮件...诸如此类的代码都写到新建用户的业务逻辑后面，会加大耦合度，降低可维护性，并且对于每个模块都是一个独立系统的情况，这种方式更是不可取。

对于简单的情形，观察者模式 `The Observer Pattern` 就足够了。如果系统中有很多地方都需要收发消息，那么它就不适用了。否则会造成类数量的膨胀，增加类的复杂性，这时候就需要一种更集中的机制来处理这些业务逻辑。

## 什么是发布/订阅模式(PUB-SUB)

现实中，并不是所有请求都期待答复，而不期待答复，自然就没有了状态。广播听过吧？收音机用过吧？就这个意思。

发布/订阅模式定义了一种一对多的依赖关系，让多个订阅者对象同时监听某一个主题对象。这个主题对象在自身状态变化时，会通知所有订阅者对象，使它们能够自动更新自己的状态。

## 特点

* 一个订阅者可以订阅多个发布者
* 消息是会到达所有订阅者处，订阅者根据 `filter` 丢掉自己不需要的消息(`filter` 是在订阅端起作用的)
* 每个订阅者都会接收到每条消息的一个副本
* 基于推送 `push`，其中消息自动地向订阅者广播，它们无须请求或轮询主题来获得新消息

发布/订阅模式内部，有多种不同类型的订阅者。

* 非持久订阅者是临时订阅类型，它们只是在主动侦听主题时才接收消息。
* 持久订阅者将接收到发布的每条消息的一个副本，即便在发布消息，它们处于"离线"状态时也是如此。
* 另外还有动态持久订阅者和受管的持久订阅者等类型。

## 优势

* 降低了模块间的耦合度：发布者与订阅者松散地耦合，并且不需要知道对方的存在。相关操作都集中在 `Publisher` 中。
* 可扩展性强：系统复杂后，可以把消息订阅和分发机制单独作为一个模块来实现，增加新特性以满足需求

## 缺陷

与其说缺陷，不如说它设计本身就有如下特点。但不管怎么说，这种模式在逻辑上不可靠的。主要体现在：

* 发布者不知道订阅者是否收到发布的消息
* 订阅者不知道自己是否收到了发布者发出的所有消息
* 发送者不能获知订阅者的执行情况
* 没人知道订阅者何时开始收到消息

## Wisper

你可能早就看过 RailsCasts 上的 [#260 Messaging with Faye](http://railscasts.com/episodes/260-messaging-with-faye) 和 [#316 Private Pub](http://railscasts.com/episodes/316-private-pub)。但今天要来介绍的是另外一个 gem。

### 首先看看传统的利用 `callback` 的实现

`Post` 模型通过回调，与 `Feed` 模型和 `User::NotifyFollowers` 服务紧密的耦合在一起。

```ruby
# app/models/post.rb
class Post
  after_create :create_feed, :notify_followers

  def create_feed
    Feed.create!(self)
  end

  def notify_followers
    User::NotifyFollowers.call(self)
  end
end

# app/controllers/api/v1/posts_controller.rb
class Api::V1::PostsController < Api::V1::ApiController
  def create
    @post = current_user.posts.build(post_params)
    if @post.save
      render_created(@post)
    else
      render_unprocessable_entity(@post.errors)
    end
  end
end
```

### 利用 `Wisper` 的 PUB-SUB 模式

```ruby
# app/models/post.rb
class Post
  # no callbacks in the models!
end
```

**Publishers** 在对象状态改变且需要触发事件的时候发布事件。

```ruby
# app/controllers/api/v1/posts_controller.rb
# corresponds to the publisher in the previous figure
class Api::V1::PostsController < Api::V1::ApiController

  include Wisper::Publisher
  def create
    @post = current_user.posts.build(post_params)
    if @post.save
      # Publish event about post creation for any interested listeners
      publish(:post_create, @post)
      render_created(@post)
    else
      # Publish event about post error for any interested listeners
      publish(:post_errors, @post)
      render_unprocessable_entity(@post.errors)
    end
  end
end
```

**Subscribers** 仅接收它们能响应的事件。

```ruby
# app/listener/feed_listener.rb
class FeedListener
  def post_create(post)
    Feed.create!(post)
  end
end
# app/listener/user_listener.rb
class UserListener
  def post_create(post)
    User::NotifyFollowers.call(self)
  end
end
```

**Event Bus** 用来管理系统中订阅者都订阅哪些频道。

```ruby
# config/initializers/wisper.rb

Wisper.subscribe(FeedListener.new)
Wisper.subscribe(UserListener.new)
```

### 进一步，根据单一职责原则把 PUB-SUB 模式和 Service Objects 联合起来

Publisher

```ruby
# app/service/financial/order_review.rb
class Financial::OrderReview
  include Wisper::Publisher
  def self.call(order)
    if order.approved?
      publish(:order_create, order)
    else
      publish(:order_decline, order)
    end
  end
end
```

Subscribers

```ruby
# app/listener/client_listener.rb
class ClientListener
  def order_create(order)
    # can implement transaction using different service objects
    Client::Charge.call(order)
    Inventory::UpdateStock.call(order)
  end

  def order_decline(order)
    Client::NotifyDeclinedOrder(order)
  end
end
```

更多用法，请参考 [Wisper wiki](https://github.com/krisleech/wisper/wiki)。

## 后记

本文的唯一作用可能就是，能让大家在设计 `Service Objects` 时候再多思考一点。虽然到底要不要抽出 `Service Objects` 也是一个见仁见智的问题。在相关连接中我有贴出 Gourmet Service Objects 一问，另外 [Service Object: What? Why? and How?](https://ruby-china.org/topics/24780) 也请抽空读一下。

这里没有交代如何在发布/订阅模式中引入队列系统的方法，所以上面的代码你是绝对没办法直接拿去用的。

## 相关链接

* [Publish–subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)
* [发布/订阅](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85)
* [设计模式---订阅发布模式(Subscribe/Publish)](http://blog.csdn.net/tjvictor/article/details/5223309)
* [The Publish-Subscribe Pattern on Rails: An Implementation Tutorial](http://www.toptal.com/ruby-on-rails/the-publish-subscribe-pattern-on-rails)
* [The Publish-Subscribe Pattern on Rails: An Implementation Tutorial](http://www.toptal.com/ruby-on-rails/the-publish-subscribe-pattern-on-rails)
* [Gourmet Service Objects](http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html)
* [WHISPER RUBY](https://shcatula.wordpress.com/2013/06/02/whisper-ruby/)
* [Getting Hexagonal with Wisper, a listener Framework for Ruby](http://devblog.reverb.com/post/57704562313/getting-hexagonal-with-wisper-a-listener)
* [Using Wisper to Decompose Applications](http://www.sitepoint.com/using-wisper-to-decompose-applications/)

