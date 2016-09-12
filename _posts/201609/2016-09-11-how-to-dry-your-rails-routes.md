---
layout: post
title:  "How to DRY Your Rails Routes"
date:   2016-09-11 10:30:00
categories: rails
tags: tip
author: "Victor"
---

工作中经常会遇到一些资源之间要共享路由，就像 controller 和 model 的 concern 文件夹一样，Rails 4 开始对 routes 也提供了 concern 功能，而且用起来很简单。

```ruby
concern :printable do
  member do
    post :print
  end
end

resources :pamphlets, concerns: :printable

resources :posters, concerns: :printable do
  collection do
    post :bulk_print
  end
end
```

看起来还不错，现在有一个需求，需要确保在浏览器的 JavaScript 禁用的情况下也能删除资源。

首先要给所有的 resources 都添加两个自定义的路由方法。我们可以这么写：

```ruby
resources_types = %i(banners cities notifications categories expresses promotions)
resources_types.each do |type|
  resources type do
    get :delete
    delete :delete, action: :destroy
  end
end
```

这样写真的很麻烦，渐渐这个数组就会越来越长，而且有些还需要添加一些其它的路由。

这时候更好的办法是在 `config/initializers` 中添加一个 delete_resource_route.rb

```ruby
module DeleteResourceRoute
  def resources(*args, &block)
    super(*args) do
      yield if block_given?
      member do
        get :delete
        delete :delete, action: :destroy
      end
    end
  end
end

ActionDispatch::Routing::Mapper.send(:include, DeleteResourceRoute)
```

添加 `app/views/application/delete.html.erb`。

```erb
<%= form_tag request.url, method: :delete do %>
  <h2>Are you sure you want to delete this record?</h2>
  <p>
    <%= submit_tag "Destroy" %>
    <% if request.referrer.present? %>
      or <%= link_to "cancel", request.referer %>
    <% end %>
  </p>
<% end %>
```

注意 Rails 的查找路径，application 是一个很好的放置共享文件的地方，过去我都是放在 shared 文件夹下，但是[文档](http://edgeguides.rubyonrails.org/layouts_and_rendering.html)其实早就提过这个问题。

> This makes app/views/application/ a great place for your shared partials

## 阅读

* [Destroy without JavaScript](http://railscasts.com/episodes/77-destroy-without-javascript-revised)
