---
layout: post
title:  "让 Content-Type 默认使用 application/json"
date:   2018-06-05 12:00:00

categories: rails
tags: api
author: "Victor"
---

注意，如果你的 API 需要直接跟微信支付对接，可能这个方法并不合适，因为微信回调用的是 XML 格式。

## 让 `Content-Type` 默认使用 `application/json`

在使用 `rails new --api` 生成项目的时候，通常意味着我们想搞一个 JSON API 项目了。

所以当客户端执行

```bash
curl -XPOST localhost:3000/email_authentication -d '{"email": "tomas"}'
```

实际上是希望执行

```bash
curl -XPOST localhost:3000/email_authentication -d '{"email": "tomas"}' -H 'Content-Type: application/json'
```

可是 Rails 默认的 `Content-Type` 默认使用  `application/x-www-form-urlencoded`。

你想通过一些小技巧来改变默认的 `Content-Type`。

```ruby
# app/controllers/application_controller.rb
 class ApplicationController < ActionController::Base
    respond_to :json
    before_action :set_default_response_format

    private

    def set_default_response_format
      request.format = :json
    end
end
```

**但是这个技巧对 Rails API 项目无效。**

我们只能通过创建 middleware 的方式来修改 Rails 默认的 `Content-Type` 了。

```ruby
# config/application.rb
# ...
require './lib/middleware/consider_all_request_json_middleware'
# ...

module MyApplication
  # ...
  class Application < Rails::Application
    # ...
    config.middleware.insert_before(ActionDispatch::Static,ConsiderAllRequestJsonMiddleware)
    # ...
```

```ruby
# lib/middleware/consider_all_request_json_middleware.rb
class ConsiderAllRequestJsonMiddleware
  def initialize app
    @app = app
  end

  def call(env)
    if env["CONTENT_TYPE"] == 'application/x-www-form-urlencoded'
      env["CONTENT_TYPE"] = 'application/json'
    end
    @app.call(env)
  end
end
```

## 原文链接

* [Content-Type application/json by default in Rails 5
](https://blog.eq8.eu/til/content-type-applicationjson-by-default-in-rails-5.html)
