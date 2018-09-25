---
layout: post
title:  "Rails 的 MessageVerifier 开发无状态的令牌功能"
date:   2018-09-24 12:00:00

categories: rails
tags: advanced
author: "Victor"
---

## 问题

许多 web 应用程序需要向用户发送链接，以允许用户在不登录的情况下执行操作。常见的例如密码重置和确认电子邮件地址之类功能。这些链接需要携带识别信息，以便服务器能够判断哪个用户正在执行操作。如果是敏感操作，链接还应该进行模糊处理，以免被人猜到网址生成算法。

我们有几种方法来生成这样的链接。比如大家都熟悉的 `has_secure_token`，这个类库包含了生成和保存令牌入库的逻辑。

在服务器上保存令牌的缺点也显而易见。以明文形式存储令牌意味着，如果攻击者能够拿到数据库，所有存储的令牌都会暴露出来。当然给令牌进行 Hashing (或 digesting) 令其不对称，算修复了这个问题，但我们仍要持久化并保护这些数据。

## 解决方案

更好的一种方式是根本不保存这些令牌。可以生成一个令牌，通过链接发送出去，然后在它被发送回服务端时进行验证。

我们可以通过编码和签名令牌实现这种无状态方案。Rails 提供了 [ActiveSupport::MessageEncryptor](https://api.rubyonrails.org/classes/ActiveSupport/MessageEncryptor.html) 和 [ActiveSupport::MessageVerifier](https://api.rubyonrails.org/classes/ActiveSupport/MessageVerifier.html) 正适合干这事。

下面以用户确认订阅邮件列表的案例来讲解该功能。

### Let’s look at some code

```ruby
class SubscriptionsController < ApplicationController
  def create
    subscription = Subscription.build(email: subscription_params[:email])

    if subscription.save
      SubscriptionsMailer.confirm_subscription(subscription.id).deliver_later
      redirect_to subscriptions_path
    else
      redirect_to subscriptions_path
    end
  end

  def confirmations
    subscription = Subscription.verify_token_and_find(token: params[:token])

    if subscription.present?
      subscription.touch(:confirmed_at)
      flash[:notice] = "Thanks for subscribing!"
    else
      flash[:error] = "Oh no! Your token is not valid."
    end

    redirect_to subscriptions_path
  end

  private

  def subscription_params
    params.require(:subscription).permit(:email)
  end
end
```

```ruby
class Subscription < ApplicationRecord
  CONFIRMATION_IN_DAYS = 7
  scope :confirmed, lambda { where.not(confirmed_at: nil) }
  validates :email, uniqueness: true

  def generate_token(expiration_in_days: CONFIRMATION_IN_DAYS)
    expiration = Time.current + expiration_in_days.days
    token_values = { id: id, expiration: expiration }
    self.class.encryptor.encrypt_and_sign(token_values)
  end

  def self.verify_token_and_find(token:)
    begin
      data = encryptor.decrypt_and_verify(token)
      given_expiration = data[:expiration]

      if given_expiration < Time.current
        return nil
      end

      find(data[:id])
    rescue ActiveSupport::MessageVerifier::InvalidSignature
      nil
    end
  end

  def self.encryptor
    key_password_for_mailing_list = ENV.fetch('KEY_PASSWORD_FOR_SUBSCRIPTION')
    SUBSCRIPTION_ENCRYPTOR_KEY = ActiveSupport::KeyGenerator
      .new(key_password_for_mailing_list)
      .generate_key(salt, 32)

    ActiveSupport::MessageEncryptor.new(SUBSCRIPTION_ENCRYPTOR_KEY)
  end
end
```

Mail 类除了调用 `subscription.generate_token` 之外也都很简单。

```ruby
class SubscriptionsMailer < ActionMailer::Base
  def confirm_subscription(subscription_id)
    subscription = Subscription.find(subscription_id)
    @link = confirmation_url(subscription: subscription)

    mail(
      from: "support@yourwebsite.com",
      subject: "Please Confirm Your Subscription",
      to: subscription.email
    ) do |format|
      format.text
      format.html
    end
  end

  private

  def confirmation_url(subscription:)
    subscriptions_url(token: subscription.generate_token)
  end
end
```

### 缺点

你需要管理另外一个密钥 `SUBSCRIPTION_ENCRYPTOR_KEY`，不过好在可以用环境变量的方式来处理。

## 原文

* [Using Rails’ MessageVerifier for Stateless Token Management](http://blog.simontaranto.com/post/2017-10-01-using-rails-messageverifier-for-stateless-token-management.html/)
