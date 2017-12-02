---
layout: post
title:  "Active Job 进阶"
date:   2017-11-30 12:30:00
categories: rails
tags: rails5
author: "Victor"
---

## 常见场景

有很多常见的设计模式和任务场景会需要用到队列。如果一个任务不需要现在立刻返回处理结果或者处理需要很长时间，那就说明需要用队列来处理任务了。

如果遇到需要很长时间执行，但是又必须立即返回结果给用户的话，一般的情况是仍然把任务加入队列，但是给用户一个进度条提示正在处理。

### Sending email

经典场景之一就是发邮件，我们都是通过后台任务处理它。Active Job 和 ActionMailer 可以很简单的结合在一起。

你只需要把 `deliver_now` 改成 `deliver_later`，Active Job 就会自动通过异步的方式把发邮件的任务放进队列。

```ruby
UserMailer.welcome(@user).deliver_later
```

### Processing images

如果需要为上传的图片生成很多不同尺寸的缩略图的话也很浪费时间，好在常用的图片处理 gem [Paperclip](https://github.com/thoughtbot/paperclip) 和 [CarrierWave](https://github.com/carrierwaveuploader/carrierwave) 都为此提供了相应的扩展。分别是 [Delayed::Paperclip](https://github.com/jrgifford/delayed_paperclip) 和 [CarrierWave Backgrounder](https://github.com/lardawge/carrierwave_backgrounder)。

用法也很简单相关 gem 的主页上说的很清楚了，不过实际使用中，目前都是使用三方的图片处理服务来做这些事。所以这块可以忽略了。

### User uploaded content

有时候用户上传的内容需要被处理，可能是 CSV 或者需要生成缩略图的视频。如果这个处理时间过长，浏览器会返回链接超时的错误。所以上传和处理文件的过程也可以放到后台队列中。

1. 接受文件并上传到亚马逊之类的文件存储服务器。
2. 添加一个任务去处理该文件。
3. 用户立刻看到一个成功页面，以便知道自己的上传成功了。
4. 后台会下载该文件，对其进行处理，并将其标记为已处理。

需要注意的是，为了改善用户体验，可以在数据库中存储这一过程的报告。其中包括刚才无法处理的错误的数据，然后再把这些错误记录单独拿出来，创建一个文件提供给用户下载。

### Generating reports

生成大型报告通常需要很长的时间，而用户可不希望一直等着。没准我们也有另外一台服务器专门处理相关任务。我们可以把该任务放到队列中，然后通过电子邮件向用户发送链接，以便在准备就绪后可以下载报告。

生成报告的流程如下：

1. 通过过滤器让用户选择他希望为哪些项目生成报告。
2. 将生成报告的任务添加到队列。
3. 让用户看到一个页面或通知，以便他们知道刚才生成报告的请求已经被处理，处理完成后用户如何获取该报告。
4. 用户获得通知，知道报告生成完毕可以直接点击下载，或者收到带有链接的邮件可以下载报告。

### Talking with external APIs

外部 API 可能很脆弱，说不准啥时候就挂了。为了不影响用户体验，这东西必须放到后台队列中执行。

有一个需求是根据当前的 IP 利用 [telize](http://www.telize.com/) API 获取一些 geo 信息。

首先我们创建一条任务，把当前 IP 传递过去。

```ruby
LogIpAddressJob.perform_later(request.remote_ip)
```

Job 类接受 IP 地址，如果该值是 `::1(localhost)` 就改为默认值 `66.207.202.15` 进行测试，然后调用 `LogIpAddress` 来完成实际工作。

```ruby
class LogIpAddressJob < ActiveJob::Base
  queue_as :default

  def perform(ip)
    ip = "66.207.202.15" if ip == "::1"
    LogIpAddress.log(ip)
  end
end
```

以下代码是最后完成具体工作的代码。

```ruby
class LogIpAddress

  def self.log(ip)
    self.new(ip).log
  end

  def initialize(ip)
    @ip = ip
  end

  def get_geo_info
    HTTParty.get("http://www.telize.com/geoip/#{@ip}").parsed_response
  end

  def log
    geo_info = get_geo_info
    Rails.logger.debug(geo_info)
    # log response to database
  end

end
```

下面你控制台看到的日志

```bash
[ActiveJob] Enqueued LogIpAddressJob (Job ID: 839db962-28a0-4e9d-9168-b08674ba192f) to Inline(default) with arguments: "::1"
[ActiveJob] [LogIpAddressJob] [839db962-28a0-4e9d-9168-b08674ba192f] Performing LogIpAddressJob from Inline(default) with arguments: "::1"
[ActiveJob] [LogIpAddressJob] [839db962-28a0-4e9d-9168-b08674ba192f] {"longitude"=>-79.4167, "latitude"=>43.6667, "asn"=>"AS21949", "offset"=>"-4", "ip"=>"66.207.202.15", "area_code"=>"0", "continent_code"=>"NA", "dma_code"=>"0", "city"=>"Toronto", "timezone"=>"America/Toronto", "region"=>"Ontario", "country_code"=>"CA", "isp"=>"Beanfield Technologies Inc.", "postal_code"=>"M6G", "country"=>"Canada", "country_code3"=>"CAN", "region_code"=>"ON"}
[ActiveJob] [LogIpAddressJob] [839db962-28a0-4e9d-9168-b08674ba192f] Performed LogIpAddressJob from Inline(default) in 572.39ms
```

### Notifying others of changes

当用户创建一条新内容的时候，我们需要通知关注他的其它用户们。如果关注者稍多的话，这一过程将消耗大量时间。

在控制器中如果创建 Tweet 成功，就创建一条任务去通知其它用户。

```ruby
def create
  @tweet = Tweet.new(tweet_params)

  respond_to do |format|
    if @tweet.save
      TweetNotifierJob.perform_later(@tweet)
      format.html { redirect_to @tweet, notice: 'Tweet was successfully created.' }    
      format.json { render :show, status: :created, location: @tweet }
    else
      format.html { render :new }
      format.json { render json: @tweet.errors, status: :unprocessable_entity }
    end
  end
end
```

Job 类中，我们可以简单的把对象传递给另外一个专门的类去处理。

```ruby
class TweetNotifierJob < ActiveJob::Base
  queue_as :default

  def perform(tweet)
    TweetNotifier.new(tweet).notify
  end
end
```

TweetNotifier 类负责具体的通知工作。

```ruby
class TweetNotifier

  def initialize(tweet)
    @tweet = tweet
  end

  def notify
    notify_mentions
    notify_followers
  end

  private

    def notify_mentions
      # search for @ mentions and notify users
    end

    def notify_followers
      # add tweet to timelines of user's followers
    end

end
```

## 任务的重试

### 目前遇到的问题

Active Job 在任务重试的功能上还比较弱，它仅提供了 `retry_job` 参数。

```ruby
class RetryJob < ApplicationJob
  queue_as :default

  rescue_from(StandardError) do
    retry_job(wait: 5.minutes)
  end

  def perform(*args)
    # Do something later
  end
end
```

这代码看起来没啥毛病，一旦任务执行出错了，那就重新排队，5分钟之后再次执行。

但是如何限制重试几次呢？如果这个任务每次都失败那就要永远不停的重试下去。

虽然能找到一个相关的 gem [ActiveJob::Retry](https://github.com/isaacseymour/activejob-retry)，但是作者自己都说他的代码不能用到生产环境。

### 我们究竟想搞啥

其实我们无非是想设置一个重试的次数，知道现在重试了几次，一旦发现超过我们设置的次数上限就停止再次尝试。

1. Setting the number of retry limit
2. Finding out the attempt number
3. Checking whether the retry limit is exceeded or not

```ruby
class LimitedRetryJob < ApplicationJob
  queue_as :default
  retry_limit 5

  rescue_from(StandardError) do |exception|
    raise exception if retry_limit_exceeded?
    retry_job(wait: attempt_number**2)
  end

  def perform(*args)
    # Do something later
  end
end
```

### 如何实现

很幸运，通过阅读 [ActiveJob::Core 的文档](http://api.rubyonrails.org/classes/ActiveJob/Core.html)，我们发现只要通过重载 ` serialize` 和 `deserialize` 就可以为被序列化的任务对象多携带一些实例变量。

官方例子如下：

```ruby
class DeliverWebhookJob < ActiveJob::Base
  def serialize
    super.merge('attempt_number' => (@attempt_number || 0) + 1)
  end

  def deserialize(job_data)
    super
    @attempt_number = job_data['attempt_number']
  end

  rescue_from(TimeoutError) do |exception|
    raise exception if @attempt_number > 5
    retry_job(wait: 10)
  end
end
```

为了实现我们的目标，可以改成如下这样：

```ruby
class ApplicationJob < ActiveJob::Base
  DEFAULT_RETRY_LIMIT = 5

  attr_reader :attempt_number

  class << self
    def retry_limit(retry_limit)
      @retry_limit = retry_limit
    end

    def load_retry_limit
      @retry_limit || DEFAULT_RETRY_LIMIT
    end
  end

  def serialize
    super.merge("attempt_number" => (@attempt_number || 0) + 1)
  end

  def deserialize(job_data)
    super
    @attempt_number = job_data["attempt_number"]
  end

  private

  def retry_limit
    self.class.load_retry_limit
  end

  def retry_limit_exceeded?
    @attempt_number > retry_limit
  end
end
```

很好，现在我们的任务基类多出了如下功能：

1. `ApplicationJob.retry_limit` - 设置任务重试的次数上限
2. `ApplicationJob#attempt_number` - 现在已经重试了几次
3. `ApplicationJob#retry_limit_exceeded?` - 检查是否超出重试上限

### 在产品环境中使用

因为并不是所有的 Job 都需要这个功能，所以最好还是单独抽出一个 module，在需要的地方 include 它。

```ruby
module ActiveJobRetryControlable
  extend ActiveSupport::Concern

  DEFAULT_RETRY_LIMIT = 5

  attr_reader :attempt_number

  module ClassMethods
    def retry_limit(retry_limit)
      @retry_limit = retry_limit
    end

    def load_retry_limit
      @retry_limit || DEFAULT_RETRY_LIMIT
    end
  end

  def serialize
    super.merge("attempt_number" => (@attempt_number || 0) + 1)
  end

  def deserialize(job_data)
    super
    @attempt_number = job_data["attempt_number"]
  end

  private

  def retry_limit
    self.class.load_retry_limit
  end

  def retry_limit_exceeded?
    @attempt_number > retry_limit
  end
end
```

## 相关链接

* [How to Use Rails Active Job](https://blog.codeship.com/how-to-use-rails-active-job/)
* [How to Integrate Sidekiq With ActiveJob](http://ruby-journal.com/how-to-integrate-sidekiq-with-activejob/)
* [active_job_retry_controlable.rb](https://gist.github.com/necojackarc/2eb388bfae26c3228c9affe592a0def6)
