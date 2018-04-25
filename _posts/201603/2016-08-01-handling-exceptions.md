---
layout: post
title:  "Handling Exceptions"
date:   2016-08-01 12:00:00
categories: rails
tags: debug
author: "Victor"
---

默认情况下，当代码遇到意外情况时 Rails 会在产品环境渲染 public 文件夹下面的静态文件。

我们也可以自定义这一过程。

## 基本

```ruby
# config/environments/development.rb
config.consider_all_requests_local = false
# config/application.rb
config.action_dispatch.rescue_responses["TaxonomiesController::Forbidden"] = :forbidden
```

```ruby
class TaxonomiesController < ApplicationController
  class Forbidden < StandardError; end
  def show
    raise Forbidden, "You are not allowed to access this product."
  end
end
```

新建 public/403.html 当 Rails 接住 Forbidden 错误的时候，会自动渲染 403 页面。

`rescue_responses` 方法会把定义错误的类 `TaxonomiesController::Forbidden` 和 `:forbidden` 作为键值对设置。根据 [Module: Rack::Utils](http://www.rubydoc.info/github/rack/rack/master/Rack/Utils) 中对 `:forbidden` 标示的错误码，去渲染对应的页面。

## Dynamic Error Pages

`rescue_from` 是另外一种处理错误的方法。它的优先级高于 `config/application.rb` 的配置。

```ruby
rescue_from "Exception", with: :forbidden
# rescue_from "TaxonomiesController::Forbidden", with: :forbidden

private

def forbidden(exception)
  render text: exception.message
end
```

## Handling Exceptions With Middleware

处理错误最好的方式是使用 `Rack middleware`。使用 `rake middleware` 命令可以看到项目中已经加载的 middleware。其中 `ActionDispatch::ShowExceptions` 可以根据错误提供不同的输出。

从源码中可以看到，在 `initializer` 方法中接受一个 `exceptions_app` 参数。`exceptions_app` 是一个 Rack app 用来处理异常，默认情况下它会委托给 `PublicExceptions`。 `PublicExceptions` 也是一个 Rails 內建的 Rack app，就是它把我们的异常渲染成了一个静态页面。

如果我们想搞成动态的错误页面，就需要在配置文件中通过指定 `exceptions_app` 来定义一个自己的 Rack app 了。这里我们先不这么干，而是利用 Rails app 来处理错误页面，所以可以这么设置。

```ruby
# config/environments/development.rb
config.consider_all_requests_local = false
# config/application.rb
config.action_dispatch.rescue_responses["TaxonomiesController::Forbidden"] = :forbidden
config.exceptions_app = self.routes
```

```ruby
class TaxonomiesController < ApplicationController
  class Forbidden < StandardError; end
  def show
    raise Forbidden, "You are not allowed to access this product."
  end
end
```

为了让它工作，我们还需要配置路由文件，首先我们试着把所有错误页面都定向到首页。

```ruby
get '403', to: 'welcome#index'
```

接下来我们更进一步把错误码定位到专门的控制器上。

先修改路由

```ruby
match '(errors)/:status', to: 'errors#show', constraints: {status: /\d{3}/}, via: :all
```

```ruby
class ErrorsController < ApplicationController
  def show
    @exception = env["action_dispatch.exception"]
    respond_to do |format|
      format.html { render action: request.path[1..-1] }
      format.json { render json: {status: request.path[1..-1], error: @exception.message}}
    end
  end
end
```

```erb
<!-- app/views/errors/403.html.erb -->
<h1>Forbidden.</h1>
<% if @exception %>
  <p><%= @exception.message %></p>
<% else %>
  <p>You are not authorized to perform that action.</p>
<% end %>
```

## 相关连接

* [Handling Exceptions](http://railscasts.com/episodes/53-handling-exceptions-revised?view=asciicast)
