---
layout: post
title:  "State of Writing API Servers with Rails"
date:   2014-09-11 10:40:00
categories: rails
tags: api
author: "Victor"
---

虽然我自己在开发 API 服务的时候选用的是 Rails 配合 Grape 的组合，但是我仍然觉得下文值得你花几分钟读一下。

### Defining an "API Server"

一个纯粹的 API 服务器应该包含以下几个特征：

1. 所有的响应都是 JSON 格式
2. 用户验证使用令牌认证(tokens)
3. ```ActionView``` 移除代码库
4. ```Sprockets``` 移除代码库
5. 输出输入含有版本概念

在我的项目中，我修改了 ```config/application.rb``` 来实现 3，4 两个功能

```ruby
# require 'rails/all'
# Pick the frameworks you want:
require 'active_record/railtie'
# require 'action_controller/railtie'
require 'action_mailer/railtie'
# require 'sprockets/railtie'
require 'rake/testtask'
# require "rails/test_unit/railtie"
require "minitest/rails/railtie"
```

### Existing Tries

已经有一些好东西可以帮我们更简单的利用 Rails 来搭建 API 服务器。基本上它们所做的只是不同版本的 ```ApplicationController::Base``` 和一些其它基本功能。本文的最后推荐了一些还不错的 gem 和代码可以作为选择。

### Getting in the Door: Authentication

实现验证最主要的方法是使用一个令牌(token)。令牌通过 header(推荐) 或者标准的参数方式传递给 API。下面是原文的例子：

```ruby
module HasApiToken
  extend ActiveSupport::Concern

  included do
    before_create :generate_api_key
  end

  private
  def generate_api_key
    self.api_key ||= Digest::SHA1.hexdigest(Time.now.to_s + attributes.inspect)
  end
end
```

然后你可以在每一个请求中，进行如下操作：

```ruby
User.find_by_api_key!(request.headers['HTTP_X_API_TOKEN'] || params[:token])
```

我不知道这种模式是否需要抽取出一个 gem，因为它是在太简单且没啥可变的地方。不论如何，借用这种思路在你们团队的创业初期，实现一个简单的验证系统足够了。

### Getting it Done: Generating JSON

你可以使用多种方式来实现这个功能。比如 ```JBuilder```, ```Rabl```, ```ActiveModel::Serializers``` 甚至自己简单的生成一个 Hash 都行。

下面是一个使用 ```ActiveModel::Serializers``` 的例子：

```ruby
class BlogPostSerializer < ActiveModel::Serializer
  attributes :title, :content, :posted_at

  has_many :comments

  def attributes
    hash = super

    # include secret if the user is an admin
    hash[:secret] = object.secret if user.admin?
    hash
  end
end
```

### Next Step: Parameter Santization and Authorization

[Strong Parameters](https://github.com/rails/strong_parameters) 对于要把验证功能移出 model 的人来说是一个良好的开端。

下面有一个样例代码：

```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributes exception because it's using mass assignment
  # without an explicit permit step.
  def create
    Person.create(params[:person])
  end

  # This will pass with flying colors as long as there's a person key in the parameters, otherwise
  # it'll raise a ActionController::MissingParameter exception, which will get caught by
  # ActionController::Base and turned into that 400 Bad Request reply.
  def update
    redirect_to current_account.people.find(params[:id]).tap do |person|
      person.update_attributes!(person_params)
    end
  end

  private
    # Using a private method to encapsulate the permissible parameters is just a good pattern
    # since you'll be able to reuse the same permit list between create and update. Also, you
    # can specialize this method with per-user checking of permissible attributes.
    def person_params
      params.required(:person).permit(:name, :age)
    end
end
```

### Action Authorization

[CanCan](https://github.com/ryanb/cancan) 不解释

### Versioning

对于这一部分，我个人的建议是使用 ```Grape``` 已经能很好的解决了。

### Documentation

可以参考 [stripe](https://stripe.com/docs/api) 的 API 文档。

* [Tomdoc](http://tomdoc.org/)
* [Swagger](http://swagger.io/)
* [slate](https://github.com/tripit/slate)

### Sharing is Caring

下面是原文作者一个基本的 API controller：

```ruby
# A Basic API Controller for Rails
# Handles authentication via Headers, params, and HTTP Auth
# Automatically makes all requests JSON format
#
# Written for production code
# Made public for: http://broadcastingadam.com/2012/03/state_of_rails_apis
#
# Enjoy!

class ApiController < ApplicationController
  class InvalidAppToken < RuntimeError ; end
  class InvalidUserToken < RuntimeError ; end

  USER_ID_HEADER = 'HTTP_X_USER_ID'
  APP_ID_HEADER = 'HTTP_X_APP_ID'

  respond_to :json

  rescue_from InvalidUserToken, InvalidAppToken do
    render :text => "Could not authenticate user or app", :status => :unauthorized
  end

  rescue_from ::CanCan::AccessDenied do
    render :text => "You do not have access to this service", :status => :forbidden
  end

  before_filter :set_default_format

  def current_user
    begin
      @current_user ||= User.find(user_id)
    rescue Mongoid::Errors::DocumentNotFound
      raise InvalidAppToken
    end
  end

  def current_app
    begin
      @current_app ||= current_user.apps.find(app_id)
    rescue Mongoid::Errors::DocumentNotFound
      raise InvalidUserToken
    end
  end

  def current_ability
    @current_ability ||= Ability.new current_app
  end

  private
  def user_id
    if params[:user_id]
      params[:user_id]
    elsif request.headers[USER_ID_HEADER]
      request.headers[USER_ID_HEADER]
    else
      authenticate_with_http_basic do |user, pass|
        user
      end
    end
  end

  def app_id
    if params[:app_id]
      params[:app_id]
    elsif request.headers[APP_ID_HEADER]
      request.headers[APP_ID_HEADER]
    else
      authenticate_with_http_basic do |user, pass|
        pass
      end
    end
  end

  def set_default_format
    request.format = 'json'
  end
end
```

### 相关链接

* [Grape](https://github.com/intridea/grape)
* [versionist](https://github.com/bploetz/versionist)
* [Versioning your APIs](http://freelancing-gods.com/posts/versioning_your_ap_is)
* [原文链接](http://hawkins.io/2012/03/state_of_rails_apis/)
