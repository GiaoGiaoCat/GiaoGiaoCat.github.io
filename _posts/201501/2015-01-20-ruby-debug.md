---
layout: post
title:  "Debugging Ruby"
date:   2015-01-20 18:00:00
categories: ruby
tags: debug
author: "Victor"
---

写代码时候，一些简单的 Bug 基本上打印1，2个变量就能找到原因。但是如果因为不了解 gem 而产生的 Bug 就比较难发现，这时候最好是加断点来调试。

## Byebug

### 基础

* 开发环境下的 server 会自动加载 `debugger`
* `byebug` 是一个普通的方法，所以可以写如下代码 `byebug if foo == "bar"`
* 在 `debugger` 模式下，按回车会执行最后一条命令，输入 `help` 可以看到完整的命令列表
* 还可以用 `info` 查看一些具体信息

```ruby
# The display command evaluates expressions every time the debugger stops.
(byebug) info display
There are no auto-display expressions now.

# Doing a info on line gives you the line number of the current break point.
(byebug) info line
Line 8 of "/Users/bparanj/projects/blog5/app/controllers/tasks_controller.rb"
```

### 停止

* `q[uit]` 退出调试并退出程序
* `quit unconditionally`, `q!` 和上一条命令一样，但略过确认的步骤
* `kill` 使用 `kill -9` 结束当前程序

### 主要命令

* `c[ontinue] <line-number>` 继续执行代码到结束，或发现到另外一处断点
* `n[ext] <number>` steps over the current line of code (runs the whole current line)
* `s[tep] <number>` steps into the current line of code (runs the current line, stopping inside any called methods)
* `up` 返回上一层函数，`down` 进入下一层函数
* `eval` 打印变量的值
* `b[ack]t[race] — a.k.a. "w[here]"` Display stack trace.
* `irb` 进入 irb 编辑代码
* `exit` 退出 irb 并继续调试
* `h[elp] <command-name>` 帮助

```ruby
# In this case, we step into the Rails source code.
(byebug) step

[1, 10] in /Users/bparanj/.rvm/gems/ruby-2.3.0@rails5/gems/actionpack-5.0.0.beta3/lib/action_controller/metal/basic_implicit_render.rb
    1: module ActionController
    2:   module BasicImplicitRender
    3:     def send_action(method, *args)
=>  4:       super.tap { default_render unless performed? }
    5:     end
    6:
    7:     def default_render(*args)
    8:       head :no_content
    9:     end
   10:   end
```

```ruby
# The eval command is one way to print the values of variables.
(byebug) eval performed?
false
```

### 调试

### 断点和捕获点 Breakpoints and Catchpoints

命令 | 解释
---|---
`b[reak]` | 当前行增加一个断点，可以在表达式中使用 `break if foo != bar`
`b[reak] <filename>:<line-number>` | 指定文件中的某一行增加一个断点。如果文件名为空，则是当前文件。 `b myfile.rb:15 unless my_var.nil?`
`b[reak] <class>(.或#)` | 指定类的某方法开头增加一个断点 `b MyClass#my_method if my_boolean`
`info breakpoints` | 列出全部断点
`cond[ition] <number> <expression>` | 为第 number 断点增加表达式，如表达式为空，则移除该断点的全部表达式
`del[ete] <number>` | 删除某断点，或删除全不断点
`disable breakpoints <number>` | 禁用某断点，或禁用全部断点
`cat[ch] <exception> off` | 启用或禁用表达式的捕获
`cat[ch]` | 列出全部捕获
`cat[ch] off` | 删除全部捕获
`sk[ip]` | 跳过捕获

## Pry

* `pry -r ./config/enviroment`
* `cd`
* `exit`
* `ls -h`
* `ls -G map` 列出所有包含 map 的方法
* `ls -M Repo --grep ^b` 列出 Repo 模块中以 b 开头的方法
* `ls -g` List global variables
* `ls -c` List constants
* `find-method currency Spree` 列出命名空间 Spree 下所有包含 currency 的方法
* `show-source Spree::Product#bought_since` any method in your application or its gems
* `show-doc Array#in_group_of`
* `show-method all.in_group_of`
* `edit-method all.in_group_of`

## pry-byebug

在 Pry 的基础上，利用 Byebug 增加 `step, next, finish, continue`

* `step` Step execution into the next line or method. Takes an optional numeric argument to step multiple times.
* `next` Step over to the next line within the same frame. Also takes an optional numeric argument to step multiple lines.
* `finish` Execute until current stack frame returns.
* `continue` Continue program execution and end the Pry session.


## 相关资源

### byebug

* [Byebug Cheatsheet](http://fleeblewidget.co.uk/2014/05/byebug-cheatsheet/#start_byebug)
* [Byebug Cheat Sheet](https://www.cheatography.com/yarik/cheat-sheets/byebug/)
* [Debugging using ByeBug Gem in Rails 5](https://rubyplus.com/articles/3631-Debugging-using-ByeBug-Gem-in-Rails-5)
* [Screencast: Debugging With Byebug](https://www.rubypigeon.com/posts/screencast-debugging-with-byebug/)

### Pry

* [Pry Screencast](http://vimeo.com/26391171)
* [Debugging Ruby](http://railscasts.com/episodes/54-debugging-ruby-revised)
* [Pry with Rails](http://railscasts.com/episodes/280-pry-with-rails)
* [Setting up Rails or Heroku to use Pry](https://github.com/pry/pry/wiki/Setting-up-Rails-or-Heroku-to-use-Pry)
