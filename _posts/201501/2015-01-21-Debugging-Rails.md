---
layout: post
title:  "Debugging Rails"
date:   2015-01-21 20:00:00
categories: rails
tags: debug
author: "Victor"
---

## Debugging Rails With Built-in Tools

### Ruby compiler checks

``ruby -c myfile.rb`` 检查 Ruby 语法错误，有些语法错误会造成程序无法运行，都可以用这个命令检查出来。

不用担心，它不会真的执行你的内部命令，仅仅是检查而已。所以不用担心自己的方法被执行而造成什么破坏（比如更改对象状态）。

### Gemfile.lock

这里包含了项目中用到的 gem 的列表，我们在 Gemfile 中引入的每一个 gem 都会引入很多其它依赖的 gems。

通常我们不用关注这个文件，除非当你依赖的 gem 因为版本问题而产生了 bug 我们需要手动指定依赖的 gems 时候才修改这个文件。

### Middleware lister

Rack middleware 会拦截每一个请求和改变响应的内容。在调试阶段，有时候我们需要知道都有哪些 middleware 被增加进来，并且它们的顺序时什么。

``rake middleware`` 可以帮我们确认目前项目中已经引入的 middleware。

### View application routes

列出当前应用响应的 URLs，已经每个 URL 接受的参数。并且它会请求哪个 controller 和哪个 action 是很有帮助的。

可以使用 ``rake routes`` 或者 ``http://localhost:3000/rails/info/routes`` 来查看。

### SQL database access

我们经常需要手动执行一些 SQL 命里来帮助我们进行调试，比如说我们要伪造一些数据或者查看一下真实的数据为什么和我们查询的不同。

``rails dbconsole`` 在这时候可以帮忙，它帮助我们直接进入数据库交互控制台。

### List available rake tasks

``rake -T`` 命里可以列举出 rails 内建的所有交互命令，以及 gems 和我们自己添加的交互命令。

### better_errors/frames

* [Better error page for Rack apps](https://github.com/charliesome/better_errors)
* [binding_of_caller](https://github.com/banister/binding_of_caller)

### Rails Debugging with Pry

使用 ``pry-byebug`` 替换 rails 默认的 console。详见[Debugging Ruby](/ruby/ruby-debug/) 中 pry 部分。

