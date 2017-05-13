---
layout: post
title:  "Mocking in Ruby with Minitest"
date:   2017-05-12 11:30:00
categories: ruby
tags: testing
author: "Victor"
---

在做单元测试的时候，如果要测试的方法需要引用外部依赖的对象，比如：发送邮件，网络通讯，记录Log, 文件系统之类的。而我们没法控制这些外部依赖的对象。为了解决这个问题，我们需要用到 `Stub` 和 `Mock` 来模拟这些外部依赖的对象。

## 基本概念

### Dummies

`dummy` 是最简单的概念。当我们在写测试的时候，为了避免 `ArgumentError` 错误，需要给方法传递一些并不真实使用的对象，通常都是使用 `Object.new` 来模拟。

### Stub

`stub` 跟 `dummy` 很像，但是 `stub` 的优势是可以响应方法调用，其实现一般会硬编码一些输入和输出。`stub` 存在的意图是为了让测试对象可以正常的执行。

`stub` 也即“桩”，很早就有这个说法了，主要出现在集成测试的过程中，从上往下的集成时，作为下方程序的替代。作用如其名，就是在需要时，能够发现它存在，即可。就好像点名，“到”即可。

### Mock

`mock` 除了有 `stub` 的功能之外，还可深入的模拟对象之间的交互方式，如：调用了几次、在某种情况下是否会抛出异常。

`mock`，主要是指某个程序的傀儡，也即一个虚假的程序，可以按照测试者的意愿做出响应，返回被测对象需要得到的信息。也即是要风得风、要雨得雨、要返回什么值就返回什么值。

## 示例

```ruby
rails g model user name:string
rails g model subscription expires_at:date user:references
```

```ruby
class User < ApplicationRecord
  has_one :subscription
end

# app/services/subscription_service.rb
class SubscriptionService
  SUBSCRIPTION_LENGTH = 1.month

  def initialize(user)
    @user = user
  end

  def apply
    if Subscription.exists?(user_id: @user.id)
      extend_subscription
    else
      create_subscription
    end
  end

  private

  def create_subscription
    subscription = Subscription.new(
      user: @user,
      expires_at: SUBSCRIPTION_LENGTH.from_now
    )

    subscription.save
  end

  def extend_subscription
    subscription = Subscription.find_by user_id: @user.id

    subscription.expires_at = subscription.expires_at + SUBSCRIPTION_LENGTH
    subscription.save
  end
end
```

```ruby
# test/services/subscription_service_test.rb
require 'test_helper'
class SubscriptionServiceTest < ActiveSupport::TestCase
  test '#create_or_extend new subscription' do
    user = users :no_sub
    subscription_service = SubscriptionService.new user
    assert_difference 'Subscription.count' do
      assert subscription_service.apply
    end
  end

  test '#create_or_extend existing subscription' do
    user = users :one
    subscription_service = SubscriptionService.new user
    assert_no_difference 'Subscription.count' do
      assert subscription_service.apply
    end
  end
end
```

### Stub

```ruby
# test/models/user_test.rb
require 'test_helper'
class UserTest < ActiveSupport::TestCase
  test '#apply_subscription' do
    mock = Minitest::Mock.new
    def mock.apply; true; end

    SubscriptionService.stub :new, mock do
      user = users(:one)
      assert user.apply_subscription
    end
  end
end

# app/models/user.rb
class User < ApplicationRecord
  has_one :subscription

  def apply_subscription
    SubscriptionService.new(self).apply
  end
end
```

### Mocking

```ruby
# test/models/user_test.rb

require 'test_helper'

class UserTest < ActiveSupport::TestCase
  test '#apply_subscription' do
    mock = Minitest::Mock.new
    mock.expect :apply, true

    SubscriptionService.stub :new, mock do
      user = users(:one)
      assert user.apply_subscription
    end

    assert_mock mock # New in Minitest 5.9.0
    assert mock.verify # Old way of verifying mocks
  end
end
```


### 参考

* [An example of using MiniTest::Mock](https://gist.github.com/adamsanderson/746155)
* [Mocking in Ruby with Minitest](https://semaphoreci.com/community/tutorials/mocking-in-ruby-with-minitest)
* [Test Doubles: in theory, in Minitest and in RSpec](http://ieftimov.com/test-doubles-theory-minitest-rspec)
* [Nobody told me Minitest was this fun](http://blog.plataformatec.com.br/2015/05/nobody-told-me-minitest-was-this-fun/)
