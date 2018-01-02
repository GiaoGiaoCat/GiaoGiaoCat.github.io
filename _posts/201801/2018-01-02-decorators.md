---
layout: post
title:  "Decorators"
date:   2018-01-02 11:30:00
categories: rails
tags: refactoring
author: "Victor"
---

## 什么是装饰模式

Decorator 是一种设计模式，它在不必改变原类文件和使用继承的情况下，动态的扩展一个对象的功能。

在 Rails 中，这种模式意味着我们可以在 `@user` 对象从 Controller 传入 View 的过程中，给其添加一些附加行为。但是在其它环境下，比如 console 或 rake 任务中， User 类的实例并不具有这些功能。这有助于我们分离代码，并在适当的时机为对象添加功能。

你可以简单的认为 Decorators 是用来清理视图中的逻辑代码并减轻 model 体积的工具。如果 model 里的某些方法仅在视图中使用，那就可以考虑移动到 decorators 中。

## 为什么需要它

视图中的逻辑很难测试，很难阅读。以下面的代码为例

```html
<h1>Show user</h1>

<dl class="dl-horizontal">
  <% if @user.public_email %>
    <dt>Email:</dt>
    <dd><%= @user.email %></dd>
  <% else %>
    <dt>Email Unavailable:</dt>
    <dd><%= link_to 'Request Email', '#', class: 'btn btn-default btn-xs' %></dd>
  <% end %>

  <dt>Name:</dt>
  <dd>
    <% if @user.first_name || @user.last_name %>
      <%= "#{ @user.first_name } #{ @user.last_name }".strip %>
    <% else %>
      No name provided.
    <% end %>
  </dd>

  <dt>Joined:</dt>
  <dd><%= @user.created_at.strftime("%A, %B %e") %></dd>

  <!-- ... -->

</dl>
```

## 使用 draper 重构

```bash
gem 'draper'

$ bundle install
$ rails generate decorator User
```

```ruby
# app/decorators/user_decorator.rb
class UserDecorator < Draper::Decorator
  delegate_all

  def email_or_request_button
    public_email ? email : h.link_to('Request Email', '#', class: 'btn btn-default btn-xs').html_safe
  end

  def full_name
    if first_name.blank? && last_name.blank?
      'No name provided.'
    else
      "#{ first_name } #{ last_name }".strip
    end
  end

  def joined_at
    created_at.strftime("%B %Y")
  end
end
```

在控制器中需要让 `ActiveRecord` 对象调用 `.decorate` 方法。

```ruby
class UsersController < ApplicationController
  before_action :do_stuff

  # GET /users
  # GET /users.json
  def index
    @users = User.all.decorate
  end

  # GET /users/1
  # GET /users/1.json
  def show
    @user = User.find(params[:id]).decorate
  end
end
```

重构后的代码。

```html
<h1><%= @user.full_name %></h1>

<dl class="dl-horizontal">
  <dt><%= @user.email_attr_text %></dt>
  <dd><%= @user.email_or_request_button %></dd>

  <dt>Name:</dt>
  <dd><%= @user.full_name %></dd>

  <dt>Joined:</dt>
  <dd><%= @user.joined_at %></dd>

  <!-- ... -->
```

也可以在重构之前撸点测试。

```ruby
require 'spec_helper'

describe UserDecorator do

  let(:first_name)  { 'John'  }
  let(:last_name)  { 'Smith' }

  let(:user) { FactoryGirl.build(:user,
                                 first_name: first_name,
                                 last_name: last_name) }

  let(:decorator) { user.decorate }

  describe '.fullname' do

    #...

    context 'with a first and last name' do

      it 'should return the full name' do
        expect(decorator.full_name).to eq("#{ first_name } #{ last_name }")
      end
    end

    context 'without a first or last name' do

      before do
        user.first_name = ''
        user.last_name = ''
      end

      it 'should return no name provided' do
        expect(decorator.full_name).to eq('No name provided.')
      end
    end

    # ...

  end
end
```

## 参考

* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Decorators on Rails](http://johnotander.com/rails/2014/03/07/decorators-on-rails/)
* [设计模式——装饰模式（Decorator）](http://blog.csdn.net/zhshulin/article/details/38665187)
* [draper](https://github.com/drapergem/draper)
