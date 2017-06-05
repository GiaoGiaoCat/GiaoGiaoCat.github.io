---
layout: post
title:  "Service Object"
date:   2017-06-03 11:30:00
categories: rails
tags: refactoring
author: "Victor"
---

### 什么是 Service Object

Service Object 没有一个固定的形态，因为它完全就是业务逻辑的封装。

### Convention over Configuration

Convention 的意义在于，它就是一个最佳实践的表现形式，Rails 本质上是一系列 web 开发中最佳实践的集合体。遵循 Convention 的 Rails 项目都长得差不多，这使得 Rails 开发者的经验能够跨项目地重用，大家可以关注在商业逻辑目标上。

### 何时使用 Service Object

因为和业务逻辑比较接近，Service Object 通常用在 Controller 中，但也可以单独使用（比如在 job ， console 或者其他 Service Object 中嵌套使用）。

* 操作逻辑很复杂。
* 操作涉及到多个 model。
* 操作涉及到调用外部服务。
* 操作不是 model 该关注的逻辑（比如定时清理过期数据）。
* 操作涉及到一系列不同的具体实现（比如用 token 认证或者 password 认证），策略模式就是干这个的。

### 基本原则 Basic principles

* does only one thing and does it well (Unix philosophy)
* can be run synchronously (i.e. blocking/in the foreground) or asynchronously (i.e. non-blocking/in the background)
* can be configured as "unique", meaning only one instance of it should be run at any time (including or ignoring parameters)
* logs all the things (start time, end time, duration, caller, exceptions etc.)
* has its own exception class(es) which all exceptions that might be raised inherit from
* does not care whether certain parameters are objects or object IDs

### 约定 Conventions

* Let your services inherit from `Services::Base`
* Let your query objects inherit from `Services::Query`
* Put your services in `app/services/`
* Use a `Services` namespace
* Give your services "verby" names,  e.g. `app/services/users/delete.rb`
* If a service operates on multiple models or no models at all, don't namespace them (Services::DoStuff) or namespace them by logical groups unrelated to models (Services::Maintenance::CleanOldStuff, Services::Maintenance::SendDailySummary, etc.)
* Some services call other services. Try to not combine multiple calls to other services and business logic in one service. Instead, some services should contain only business logic and other services only a bunch of service calls but no (or little) business logic. This keeps your services nice and modular.

## 参考

### articles

* [Service Object 整理和小结](https://ruby-china.org/topics/23892)
* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Service Objects With Rails Using Aldous](https://code.tutsplus.com/tutorials/service-objects-with-rails-using-aldous--cms-23689)

### gems

* [simple_command](https://github.com/nebulab/simple_command)
* [services](https://github.com/krautcomputing/services)
* [aldous](https://github.com/envato/aldous)
* [rectify](https://github.com/andypike/rectify)
