---
layout: post
title:  "How To Change The View Path Of A Controller In Rails"
date:   2016-09-26 10:30:00
categories: rails
tags: tip
author: "Victor"
---

## 多个控制器共享模板
根据 Rails 的建议，应该把 shared 模板放在 `view/application` 目录下，但是你也可以选择另外一个方案。

比如，我们想在另外一个控制器，使用 `app/views/posts` 的模板。


```ruby
class UserPostsController
  def self.controller_path
    "posts" # change path from app/views/user_posts to app/views/posts
  end
end

class PostsController
   ...
end
```

## 不同的子站点，使用不同的模板
如果我们的项目需要为不同的客户使用不同的模板，那么可以使用下面的技术。

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :set_view_path

  private

  def set_view_path
    case request.host
    when "mysite.com"
      site = "my_site" # root view path will be app/views/my_site
    when "myothersite.com"
      site = "my_other_site" # root view path will be app/views/my_other_site
    else
      site = "default_site" # root view path will be app/views/default_site
    end
    prepend_view_path "#{Rails.root}/app/views/#{site}"
  end
end
```

## 原文

* [How To Change The View Path Of A Controller In Rails](https://solidfoundationwebdev.com/blog/posts/how-to-change-the-view-path-of-a-controller-in-rails)
