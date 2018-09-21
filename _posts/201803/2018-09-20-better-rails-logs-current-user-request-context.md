---
layout: post
title:  "Rails 日志中记录当前用户"
date:   2018-09-20 14:00:00

categories: rails
tags: debug
author: "Victor"
---

查看日志的时候，如果能看到引起该操作的用户是哪个就会方便多了。

Rails 的 `config` 支持添加 [log_tag](https://guides.rubyonrails.org/configuring.html#rails-general-configuration)，它有一些内置的选项，比如 `subdomain` 和 `request_id` 也可以添加一些 Proc。

将以为的配置代码插入到 `config/application.rb` 就会在日志中插入一些标记，打印出 `session` 中的 `user_id`。

```ruby
config.log_tags = [
  :request_id,
  ->(req) {
    session_key = (Rails.application.config.session_options || {})[:key]
    session_data = req.cookie_jar.encrypted[session_key] || {}
    user_id = session_data["user_id"] || "guest"
    "user: #{user_id.to_s}"
  }
]
```

之后日志看起来就是这样：

```
[some-request-id] [user: 123] ...
[some-request-id] [user: guest] ...
```

如果需要更多的上下文，可以看一下 https://github.com/remind101/request_id 应该也很有用。

原文看这里 [Better Rails Logs: Current User Request Context](http://blog.simontaranto.com/post/2018-05-18-better-rails-logs-current-user-request-context.html/)。
