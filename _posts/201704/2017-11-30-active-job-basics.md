---
layout: post
title:  "Active Job 基础"
date:   2017-11-30 11:30:00
categories: rails
tags: rails5
author: "Victor"
---

## 简介

开发中涉及到调用三方服务API，运行时间长，结果不需要实时反馈给用户这样的任务，都可以使用异步处理。常见的场景包括：发邮件和短信、图片处理、定时清理等、爬虫。

Active Job 提供了统一的接口，后端可以随意选择 `Delayed Job`，`Resque`，`Sidekiq`。

## 用法

### 创建作业

```bash
# 创建作业
rails generate job guests_cleanup

# 创建在指定队列中运行的作业
rails generate job guests_cleanup --queue urgent
```

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default

  def perform(*guests)
    # 稍后做些事情
  end
end
```

### 入队作业

`set` 支持可选参数：`:wait、:wait_until、:queue、:priority`，它的具体实现由 ConfiguredJob 完成，主要是处理各个参数，起到配置作用。

```bash
# 入队作业，作业在队列系统空闲时立即执行
GuestsCleanupJob.perform_later guest

# 入队作业，在明天中午执行
GuestsCleanupJob.set(wait_until: Date.tomorrow.noon).perform_later(guest)

# 入队作业，在一周以后执行
GuestsCleanupJob.set(wait: 1.week).perform_later(guest)

# `perform_now` 和 `perform_later` 会在幕后调用 `perform` 因此可以传入任意个参数
GuestsCleanupJob.perform_later(guest1, guest2, filter: 'some_filter')
```

```ruby
# `perform_now` 代码会立即执行，在这开发环境会很实用。
MyJob.new(*args).perform_now
MyJob.perform_now("mike")
```

### 执行作业

生产环境中入队和执行作业需要使用队列后端，即要为 Rails 提供一个第三方队列库。Rails 本身只提供了一个进程内队列系统 `:inline`，把作业存储在 RAM 中。如果进程崩溃，或者设备重启了，默认的异步后端会丢失所有作业。这对小型应用或不重要的作业来说没什么，但是生产环境中的多数应用应该挑选一个持久后端。

Active Job 为多种队列后端（Sidekiq、Resque、Delayed Job，等等）内置了适配器。最新的适配器列表参见 [ActiveJob::QueueAdapters 的 API 文档](http://api.rubyonrails.org/v5.1.1/classes/ActiveJob/QueueAdapters.html)。

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    # 要把适配器的 gem 写入 Gemfile
    # 请参照适配器的具体安装和部署说明
    config.active_job.queue_adapter = :sidekiq
  end
end
```

```ruby
class GuestsCleanupJob < ApplicationJob
  self.queue_adapter = :resque
  #....
end

# 现在，这个作业使用 `resque` 作为后端队列适配器
# 把 `config.active_job.queue_adapter` 配置覆盖了
```

Rails 应用中的作业并行运行，因此多数队列库要求为自己启动专用的队列服务（与启动 Rails 应用的服务不同）。启动队列后端的说明参见各个库的文档。

## 队列

多数适配器支持多个队列，默认使用的 queue_name 是 `default`。Active Job 允许把作业调度到具体的队列中：

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end
```

1. 队列名称可以使用 application.rb 文件中的 `config.active_job.queue_name_prefix` 选项配置前缀。
2. 默认的队列名称前缀分隔符是 `_`。这个值可以使用 application.rb 文件中的 `config.active_job.queue_name_delimiter` 选项修改。
3. 队列有优先级这个属性，优先级高的会被先执行。类方法 `queue_with_priority` 可以进行设置。

```ruby

# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
    config.active_job.queue_name_delimiter = '.'
  end
end

# app/jobs/guests_cleanup_job.rb
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  queue_with_priority 50

  def perform(post)
    post.to_feed!
  end
end

# 在生产环境中，作业在 production.low_priority 队列中运行
# 在交付准备环境中，作业在 staging.low_priority 队列中运行
```

如果想更进一步控制作业在哪个队列中运行，可以把 `:queue` 选项传给 `#set` 方法：

```ruby
MyJob.set(queue: :another_queue).perform_later(record)
```

如果想在作业层控制队列，可以把一个块传给 `#queue_as` 方法。那个块在作业的上下文中执行（因此可以访问 `self.arguments`），必须返回队列的名称：

```ruby
class ProcessVideoJob < ApplicationJob
  queue_as do
    video = self.arguments.first
    if video.owner.premium?
      :premium_videojobs
    else
      :videojobs
    end
  end

  def perform(video)
    # 处理视频
  end
end

ProcessVideoJob.perform_later(Video.last)
```

**确保队列后端“监听”着队列名称。某些后端要求指定要监听的队列。**

## 排队与重试

除了使用 `set` 联合 `perform_later` 的方法来入队之外，还可以使用 `enqueue` 来代替。

```ruby
SomeJob.set(wait: 10.minutes).perform_later(record)
SomeJob.new(record).enqueue(wait: 10.minutes)
```

`enqueue` 可接受的参数如下：

```ruby
my_job_instance.enqueue
my_job_instance.enqueue wait: 5.minutes
my_job_instance.enqueue queue: :important
my_job_instance.enqueue wait_until: Date.tomorrow.midnight
my_job_instance.enqueue priority: 10
```

当 `rescue_from` 抛出异常时，你可以用 `retry_job` 方法要求 Active Job 再次执行你的任务。

```ruby
class SiteScraperJob < ActiveJob::Base
  rescue_from(ErrorLoadingSite) do
    retry_job queue: :low_priority
  end

  def perform(*args)
    # raise ErrorLoadingSite if cannot scrape
  end
end
```


## 回调

Active Job 在作业的生命周期内提供了多个钩子。回调用于在作业的生命周期内触发逻辑。可用的回调:

* `before_enqueue`
* `around_enqueue`
* `after_enqueue`
* `before_perform`
* `around_perform`
* `after_perform`

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default

  before_enqueue do |job|
    # 对作业实例做些事情
  end

  around_perform do |job, block|
    # 在执行之前做些事情
    block.call
    # 在执行之后做些事情
  end

  def perform
    # 稍后做些事情
  end
end
```

## Action Mailer

最常见的作业是在 *请求 - 响应* 循环之外发送电子邮件，这样用户无需等待。Active Job 与 Action Mailer 是集成的，因此可以轻易异步发送电子邮件：

```ruby
# 如需想现在发送电子邮件，使用 #deliver_now
UserMailer.welcome(@user).deliver_now

# 如果想通过 Active Job 发送电子邮件，使用 #deliver_later
UserMailer.welcome(@user).deliver_later
```

## 国际化

创建作业时，使用 `I18n.locale` 设置。如果异步发送电子邮件，可能用得到：

```ruby
I18n.locale = :eo
UserMailer.welcome(@user).deliver_later # 使用世界语本地化电子邮件
```

## GlobalID

Active Job 支持参数使用 GlobalID。这样便可以把 Active Record 对象传给作业，而不用传递类和 ID，再自己反序列化。

```ruby
class TrashableCleanupJob < ApplicationJob
  def perform(trashable, depth)
    trashable.cleanup(depth)
  end
end
```

## 异常

Active Job 允许捕获执行作业过程中抛出的异常。

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default

  rescue_from(ActiveRecord::RecordNotFound) do |exception|
   # 处理异常
  end

  def perform
    # 稍后做些事情
  end
end
```

有了 GlobalID，可以序列化传给 `#perform` 方法的整个 Active Record 对象。

**如果在作业入队之后、调用 `#perform` 方法之前删除了传入的记录，Active Job 会抛出 `ActiveJob::DeserializationError` 异常。**

## 测试

因为自定义的作业在应用的不同层排队，所以我们既要测试作业本身（入队后的行为），也要测试是否正确入队了。

```ruby
require 'test_helper'

class BillingJobTest < ActiveJob::TestCase
  test 'that account is charged' do
    BillingJob.perform_now(account, product)
    assert account.reload.charged_for?(product)
  end
end
```

默认情况下，ActiveJob::TestCase 把队列适配器设为 `:test`，因此作业是内联执行的。此外，在运行任何测试之前，它会清理之前执行的和入队的作业，因此我们可以放心假定在当前测试的作用域中没有已经执行的作业。

Active Job 自带了很多自定义的断言，可以简化测试。可用的断言列表参见 [ActiveJob::TestHelper 模块的 API 文档](http://api.rubyonrails.org/v5.1.1/classes/ActiveJob/TestHelper.html)。

不管作业是在哪里调用的（例如在控制器中），最好都要测试作业能正确入队或执行。这时就体现了 Active Job 提供的自定义断言的用处。例如，在模型中：

```ruby
require 'test_helper'

class ProductTest < ActiveJob::TestCase
  test 'billing job scheduling' do
    assert_enqueued_with(job: BillingJob) do
      product.charge(account)
    end
  end
end
```

## 其它

* `around_enqueue、around_perform、before_enqueue、enqueue、enqueue_at、perform_start、perform` 等过程均有日志记录，默认和 Rails 应用使用同一日志系统。
* 每个任务都有全局唯一的 job_id。



## 本文摘自

[Active Job 基础](https://ruby-china.github.io/rails-guides/active_job_basics.html)
[Active Job 使用](https://book.a-bitcoin.com/activejob/active_job_shi_yong.html)
