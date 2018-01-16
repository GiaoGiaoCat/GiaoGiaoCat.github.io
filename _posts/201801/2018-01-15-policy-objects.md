---
layout: post
title:  "Policy Objects"
date:   2018-01-15 11:30:00
categories: rails
tags: refactoring
author: "Victor"
---

## 什么是策略模式

策略模式是指对一系列的算法定义，并将每一个算法封装起来，而且使它们还可以相互替换。策略模式让算法独立于使用它的客户而独立变化。

在软件开发中常常遇到类似的情况，实现某一个功能有多种算法或者策略，我们可以根据环境或者条件的不同选择不同的算法或者策略来完成该功能。如查找、排序等。

在 Rails 项目中 Policy Objects 多数情况下是用来提供相同行为的不同实现，进一步缩小限定的话就是用户可以根据权限或角色的不同，从不同策略中进行选择。

**Rails use Policy Objects for a group of business rules like an Authorizer that regulates which data a user can access.**

## 和其它模式的区别

Policy Objects 和 Service Objects 很相近。你可以简单的理解为：Policy Objects 用来负责读操作，而 Service Objects 用来负责写操作。

Policy Objects 和 Query Objects 的区别是 Query Objects 执行的是 SQL 查询并返回结果集，而 Policy Objects 执行的是已经载入内存中的 领域模型实例 的 实例方法。


## 例子

在用户创建一个 `project` 之前，Controller 会做如下检查

1. `current_user` 是否是 `manager`
2. 是否有权限创建 `project`
3. 用户当前的 `projects` 是否已经超过允许的最大数量
4. Redis 的 `key/value` 中是否存在创建 `project` 的代码块（我们把创建项目的相关参数存到了 redis 中，稍后用后台任务来创建）

```ruby
class CreateProjectPolicy
  def initialize(user, redis_client)
    @user = user
    @redis_client = redis_client
  end

  def allowed?
    @user.manager? && below_project_limit && !project_creation_blocked
  end

 private

  def below_project_limit
    @user.projects.count < Project.max_count
  end

  def project_creation_blocked
    @redis_client.get('projects_creation_blocked') == '1'
  end
end
```

```ruby
class ProjectsController < ApplicationController
  def create
    if policy.allowed?
      @project = Project.create!(project_params)
      render json: @project, status: :created
    else
      head :unauthorized
    end
  end

  private

  def project_params
    params.require(:project).permit(:name, :description)
  end

  def policy
    CreateProjectPolicy.new(current_user, redis)
  end

  def redis
    Redis.current
  end
end
```

```ruby
class User < ActiveRecord::Base
  enum role: [:manager, :employee, :guest]
end
```

## 参考

* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Policy Objects](https://makandracards.com/alexander-m/43084-policy-objects)
* [Authorizers, Extractors, and Policy objects](https://www.sethvargo.com/authorizers-extractors-and-policy-objects/)
* [Policy Object](https://medium.com/@steverob/policy-object-d158495265fa)
* [Policy Objects in Ruby on Rails](http://www.eq8.eu/blogs/41-policy-objects-in-ruby-on-rails)
