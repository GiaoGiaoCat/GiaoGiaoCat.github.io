---
layout: post
title:  "lib/ 目录下应该放什么代码"
date:   2018-01-24 17:10:00

categories: rails
tags: tip
author: "Victor"
---

团队的小伙伴对一个问题很迷惑，究竟 rails 的 `/lib` 目录下要放什么代码。

> lib – Application specific libraries. Basically, any kind of custom code that doesn’t belong under controllers, models, or helpers. This directory is in the load path.

上面这一段是官方文档的介绍，基本上看完了还是懵蔽的状态。那这里有一个简单的原则来帮你判断代码是否应该放在 `/lib` 目录下。

**libraries that are not packaged as gems but could be.**

## `/app` 目录
所有业务逻辑相关的代码都放在这里。如果你创建的 PORO (Plain-Old Ruby Object) 跟业务逻辑有关，那么也应该在 `/app` 目录下。当然根据团队的习惯，可能在 `/app` 下还有一些子目录，用来组织文件。比如：`services, observers, presenters, decorators` 等等。

### What you should put inside this folder:
* Your models, controllers, views and mailers.
* Any other entity you create (class, module) that holds some business logic.

### What you should not put inside this folder:
* Configuration, migrations, tests (obviously).
* Generic entities (class or module). For example, a class to perform a bunch of generic calculations on an array (median, etc) should be in `/lib`.


## `/lib` 目录

现在你已经知道什么应该放在 `/lib` 目录了，我再举一些例子：

* 为基础类添加的 calculations 模块
* 通用的图片转换功能
* 和三方 API 交互的类
* rails 和 ruby 的猴子补丁

## 参考

* [Rails: /app vs /lib](https://devblast.com/b/rails-app-vs-lib)
* [What code goes in the lib/ directory](https://codeclimate.com/blog/what-code-goes-in-the-lib-directory/)
