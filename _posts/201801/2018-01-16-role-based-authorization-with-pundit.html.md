---
layout: post
title:  "基于 Pundit 的权限系统"
date:   2018-01-15 11:30:00
categories: rails
tags: advanced
author: "Victor"
---

主要基于 Pundit 和 devise 实现一个基于角色的授权系统，用到了 ancestry 做树状结构。

## 模型

```ruby
class User < ApplicationRecord
  belongs_to :permission_group
  has_and_belongs_to_many :permissions
end

class Permission < ApplicationRecord
  has_and_belongs_to_many :permission_groups
  has_and_belongs_to_many :users
end

class PermissionGroup < ApplicationRecord
  has_and_belongs_to_many :permissions
end
```


## 参考

### Gems
* [TheRole 3.0](https://github.com/the-teacher/the_role)
* [Fibman](https://github.com/Warrenoo/fibman)

### 文章
* [Role-based Authorization with Pundit](http://www.xyyz.me/2015/07/24/role-based-authorization-with-pundit.html)
* [Action Controller - 控制 HTTP 流程](https://ihower.tw/rails/actioncontroller-cn.html)
* [Ruby on Rails 安全指南](https://ruby-china.github.io/rails-guides/security.html#cross-site-request-forgery-csrf)
* [A menu system with Pundit](https://www.cookieshq.co.uk/posts/a-menu-system-with-pundit)
