---
layout: post
title:  "Rails 的 CurrentAttributes"
date:   2018-10-21 12:00:00

categories: rails
tags: advanced
author: "Victor"
---

## 简介

Abstract super class that provides a thread-isolated attributes singleton, which resets automatically before and after each request. This allows you to keep all the per-request attributes easily available to the whole system.

### 不使用 CurrentAttributes

```ruby
def process_data
  processor = DataProcessor.new
  if processor.call_long_data_processing
    processor.log("#{Current.user} called data processing")
  end
end

def irrelevant_action
  render plain: 'hello'
end
```

如果 A 用户先访问处理比较慢的 `process_data` action，然后 B 用户请求了处理很快的 `irrelevant_action` action。假设，我们的服务器是单线程的，那就没啥问题，只能等 A 用户的请求返回结果之后，才会处理 B 用户的请求。

但如果我们是多线程服务，正好有空余的线程可以处理 B 用户的请求，那么 `Current.user` 就有可能被改变。

### 正确使用 CurrentAttributes

```ruby
# app/models/current.rb
class Current < ActiveSupport::CurrentAttributes
  attribute :account, :user
  attribute :request_id, :user_agent, :ip_address

  resets { Time.zone = nil }

  def user=(user)
    super
    self.account = user.account
    Time.zone    = user.time_zone
  end
end
```

```ruby
# app/controllers/concerns/authentication.rb
module Authentication
  extend ActiveSupport::Concern

  included do
    before_action :authenticate
  end

  private
    def authenticate
      if authenticated_user = User.find_by(id: cookies.encrypted[:user_id])
        Current.user = authenticated_user
      else
        redirect_to new_session_url
      end
    end
end
```

```ruby
# app/controllers/concerns/set_current_request_details.rb
module SetCurrentRequestDetails
  extend ActiveSupport::Concern

  included do
    before_action do
      Current.request_id = request.uuid
      Current.user_agent = request.user_agent
      Current.ip_address = request.ip
    end
  end
end
```

```ruby
class ApplicationController < ActionController::Base
  include Authentication
  include SetCurrentRequestDetails
end
```

```ruby
class MessagesController < ApplicationController
  def create
    Current.account.messages.create(message_params)
  end
end
```

```ruby
class Message < ApplicationRecord
  belongs_to :creator, default: -> { Current.user }
  after_create { |message| Event.create(record: message) }
end
```

```ruby
class Event < ApplicationRecord
  before_create do
    self.request_id = Current.request_id
    self.user_agent = Current.user_agent
    self.ip_address = Current.ip_address
  end
end
```

### 注意事项

* 像 `Current` 这样的全局单例类如果太多，会使得 model 很混乱。
* `Current` 应该只被用来处理少数的顶级全局变量，例如 `account, user` 和 request details。因为所有请求的所有操作都或多或少的用得上这些属性。
* **绝对不要把 controller 专用的属性放在这里。**

这里还有一个反向的观点，认为引入了类似全局变量的东西，会增加 Rails 项目代码的理解复杂度，详见 [Rails' CurrentAttributes considered harmful](https://ryanbigg.com/2017/06/current-considered-harmful)

## 相关链接

* [Scoping records to the current account in Ruby on Rails applications](https://medium.com/@afcapel/scoping-records-to-the-current-account-in-ruby-on-rails-applications-b6481e96bc9)
* [#29180](https://github.com/rails/rails/commit/24a864437e845febe91e3646ca008e8dc7f76b56)
* [Rails' CurrentAttributes considered harmful](https://ryanbigg.com/2017/06/current-considered-harmful)
* [On Writing Software Well #3: Using globals when the price is right](https://www.youtube.com/watch?v=D7zUOtlpUPw)
