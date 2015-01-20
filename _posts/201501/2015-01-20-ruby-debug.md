---
layout: post
title:  "Debugging Ruby"
date:   2015-01-20 18:00:00
categories: ruby
tags: tip
author: "Victor"
---

## 结论

最好装 ``pry-rails`` 配合 ``pry-byebug``， 前者可以让 ``pry`` 替代默认的 ``rails console``

## pry-byebug

在 Pry 的基础上，利用 Byebug 增加 ``step, next, finish, continue``

* ``step`` Step execution into the next line or method. Takes an optional numeric argument to step multiple times.
* ``next`` Step over to the next line within the same frame. Also takes an optional numeric argument to step multiple lines.
* ``finish`` Execute until current stack frame returns.
* ``continue`` Continue program execution and end the Pry session.

## Byebug

### 基础

* 开发环境下的 server 会自动加载 ``debugger``
* ``byebug`` 是一个普通的方法，所以可以写如下代码 ``byebug if foo == "bar"``
* 在 ``debugger`` 模式下，按回车会执行最后一条命令

### 停止

* ``q[uit]`` 退出调试并退出程序
* ``quit unconditionally``, ``q!`` 和上一条命令一样，但略过确认的步骤
* ``kill`` 使用 ``kill -9`` 结束当前程序

### 主要命令

* ``c[ontinue] <line-number>`` 继续执行代码到结束，
* ``n[ext] <number>`` 执行到下一行
* ``s[tep] <number>`` 执行下一样，并进入函数内部
* ``b[ack]t[race] — a.k.a. "w[here]"`` Display stack trace.
* ``irb`` 进入 irb 编辑代码
* ``exit`` 退出 irb 并继续调试
* ``h[elp] <command-name>`` 帮助

### 断点和捕获点 Breakpoints and Catchpoints

命令 | 解释
---|---
``b[reak]`` | 当前行增加一个断点，可以在表达式中使用 ``break if foo != bar``
``b[reak] <filename>:<line-number>`` | 指定文件中的某一行增加一个断点。如果文件名为空，则是当前文件。 ``b myfile.rb:15 unless my_var.nil?``
``b[reak] <class>(.或#)`` | 指定类的某方法开头增加一个断点 ``b MyClass#my_method if my_boolean``
``info breakpoints`` | 列出全部断点
``cond[ition] <number> <expression>`` | 为第 number 断点增加表达式，如表达式为空，则移除该断点的全部表达式
``del[ete] <number>`` | 删除某断点，或删除全不断点
``disable breakpoints <number>`` | 禁用某断点，或禁用全部断点
``cat[ch] <exception> off`` | 启用或禁用表达式的捕获
``cat[ch]`` | 列出全部捕获
``cat[ch] off`` | 删除全部捕获
``sk[ip]`` | 跳过捕获

## Pry

* ``pry -r ./config/enviroment``
* ``cd``
* ``exit``
* ``ls -h``
* ``show-doc Array#in_group_of``
* ``show-method all.in_group_of``
* ``edit-method all.in_group_of``

## 相关资源

* [Pry Screencast](http://vimeo.com/26391171)
* [Byebug Cheatsheet](http://fleeblewidget.co.uk/2014/05/byebug-cheatsheet/#start_byebug)
* [Debugging Ruby](http://railscasts.com/episodes/54-debugging-ruby-revised)
* [Pry with Rails](http://railscasts.com/episodes/280-pry-with-rails)
* [Setting up Rails or Heroku to use Pry](https://github.com/pry/pry/wiki/Setting-up-Rails-or-Heroku-to-use-Pry)
