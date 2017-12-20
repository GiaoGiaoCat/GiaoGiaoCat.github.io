---
layout: post
title:  "Service Object"
date:   2017-06-04 11:30:00
categories: rails
tags: refactoring
author: "Victor"
---

## Service Object with active_type

### 什么是 Service Object

Service Object 是一个普通的 Ruby 对象，用于将业务逻辑分解为可管理的类和方法。

因为和业务逻辑比较接近，Service Object 通常用在 Controller 中，但也可以单独使用（比如在 job ， console 或者其他 Service Object 中嵌套使用）。

### 何时使用 Service Object

* 操作逻辑很复杂。（在会计期结束时关闭账本）
* 操作涉及到多个 model。（一个在线商城的 purchase 动作涉及到 Order, CreditCard 和 Customer 模型）
* 操作涉及到调用外部服务。（发送短信验证码）
* 操作不是 model 该关注的核心逻辑（比如定时清理过期数据）。
* 操作涉及到一系列不同的具体实现（比如用 token 认证或者 password 认证），策略模式就是干这个的。

### Convention over Configuration

Convention 的意义在于，它就是一个最佳实践的表现形式，Rails 本质上是一系列 web 开发中最佳实践的集合体。遵循 Convention 的 Rails 项目都长得差不多，这使得 Rails 开发者的经验能够跨项目地重用，大家可以关注在商业逻辑目标上。


### 基本原则 Basic principles

* does only one thing and does it well (Unix philosophy)
* can be run synchronously (i.e. blocking/in the foreground) or asynchronously (i.e. non-blocking/in the background)
* can be configured as "unique", meaning only one instance of it should be run at any time (including or ignoring parameters)
* logs all the things (start time, end time, duration, caller, exceptions etc.)
* has its own exception class(es) which all exceptions that might be raised inherit from
* does not care whether certain parameters are objects or object IDs

## 基于 active_type 实现

### 约定 Conventions

* Let your services inherit from `ApplicationService`
* Put your services in `app/services/`
* 文件以 `service` 结尾并能描述出功能，并以动词开头。例如：`app/services/memberships/become_collaborator_service.rb`
* 文件继承 `app/services/application_service.rb`
* 所有的商业逻辑都放在 `perform` 方法中，也有人喜欢 `call, run, execute`
* Consider wrapping call methods in transactions

```ruby
class ApplicationService < ActiveType::Object
  self.abstract_class = true

  after_save :perform

  private

  def perform
    raise NotImplementedError, 'Must be implemented by subtypes.'
  end
end
```

### 例子

```ruby
# app/services/roles/become_admin_service.rb
class Roles::BecomeAdminService < ApplicationService
  attribute :forum_id, :integer
  attribute :user_id, :integer

  belongs_to :user
  belongs_to :forum

  validate :need_membership

  delegate :members, to: :forum

  private

  def perform
    return if user.has_role?(:moderator, forum)
    user.add_role(:admin, forum)
  end

  def need_membership
    errors.add :base, :need_membership unless members.include?(user)
  end
end
```

```ruby
role = Roles::BecomeAdminService.new(forum: @forum, user: @member)
role.save
```

### 优势

### Service objects 可以很清晰的体现出应用的功能

只要扫一眼 services 文件夹，看看名字就能猜出来个大概了。 `ApproveTransaction, CancelTransaction, BlockAccount, SendTransactionApprovalReminder, ApproveTransaction, SendTestNewsletter, ImportUsersFromCsv, CreateUser, AuthenticateUsingTwitter, ObfuscateCode, CompleteOrder`。 注意，起名的时候尽量以 commands or actions 的形式来命名，便于理解。

不需要关心 controller 中添加了什么特殊逻辑，model 添加了哪些 callback 和 observers，所有的商业逻辑都在 `perform` 方法中。

### 让 controller 和 model 更干净的，便于测试

现在 Controllers 只需要把携带的参数 `params, session, cookies` 传递给 service 并根据 service object 是否正确保存，做不同的响应状态就行 `redirect or render`。看起来的代码完全跟 Scaffold 生成的一样，利用 `validate` 方法可以拿到 errors 信息。

Models 也只需要关注 `associations, scopes, validations and persistence`。

```ruby
# app/services/invite/accept_invite_service.rb
class Invite::AcceptInviteService < ApplicationService
  attribute :invite
  attribute :user

  private

  def perform  
    invite.accept!(user)
    UserMailer.invite_accepted(invite).deliver
  end
end
```

```ruby
class InviteController < ApplicationController
  def accept
    invite = Invite.find_by_token!(params[:token])
    service = Invite::AcceptInviteService.new(invite: invite, user: current_user)
    if service.save
      redirect_to invite.item, notice: "Welcome!"
    else
      redirect_to '/', alert: "Oopsy!"
    end
  end
end
```

```ruby
class Invite < ActiveRecord::Base
  def accept!(user, time=Time.now)
    update_attributes!(accepted_by_user_id: user.id, accepted_at: time)
  end
end
```

### DRY 并且容易修改

保持 service objects 简单，可以把几个 service compose 在一起，方便复用。这样项目的模块化程度更高，修改起来也很容易。

```ruby
class SendTestNewsletter < ApplicationService
  attribute :newsletter

  private

  def perform
    campaign = CreateMailChimpCampaign.create(newsletter)
    DeliverTestEmail.create(campaign)
    DeleteCampaign.create(campaign) # Don't keep the test campaign around
  end

end

class SendNewsletter < ApplicationService
  attribute :newsletter

  private

  def perform
    campaign = CreateMailChimpCampaign.create(newsletter)
    DeliverCampaign.create(campaign)
    # Could easily delete here as well, but we want to retain the legit campaigns
  end
end
```

### 其它优势

* 因为 service 本身就是简单的 Ruby 对象，所以测试起来也很容易
* 你可以在任何地方使用 service object，在 `service object`, `DelayedJob / Rescue / Sidekiq`， `Rake Task`，`console`， `helper` 等等。

## 参考

### articles

* [Service Object 整理和小结](https://ruby-china.org/topics/23892)
* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Service Objects With Rails Using Aldous](https://code.tutsplus.com/tutorials/service-objects-with-rails-using-aldous--cms-23689)
* [My take on services in Rails](http://blog.sundaycoding.com/blog/2014/11/25/my-take-on-services-in-rails/)

### gems

* [business_process](https://github.com/Selleo/business_process)
* [simple_command](https://github.com/nebulab/simple_command)
* [services](https://github.com/krautcomputing/services)
* [aldous](https://github.com/envato/aldous)
* [rectify](https://github.com/andypike/rectify)
* [pattern](https://github.com/Selleo/pattern) 这个值得好好看看
