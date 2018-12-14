---
layout: post
title:  "为 Rails 的 Rake 添加 sandbox 模式"
date:   2018-11-27 12:00:00

categories: rails
tags: rake
author: "Victor"
---

有时我们在 Rails 项目中要执行一些比较复杂的操作数据库的 rake tasks。可能是更新一批记录的属性或者批量删除数据。

而 Rails 没有像 console 一样给我们提供一个 sandbox。在 sandbox 模式下，当退出 console 的时，所有数据库操作都会回滚。

现在我们把这个模式移植到 rake 中，首先创建一个 sandbox.rake 从 [console_sandbox.rb](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/railties/console_sandbox.rb) 中抄一段代码过来。

```ruby
desc 'Run rake tasks with sandboxing'
task sandbox: :environment do

  # Start transaction
  ActiveRecord::Base.connection.begin_transaction(joinable: false)
  puts "🆒 Sandbox Mode: ON 🆒"

  # Run your task
  Rake.application.invoke_task(ARGV.delete_at(1))

  # Teardown
  at_exit do
    puts "Rolling back......"
    ActiveRecord::Base.connection.rollback_transaction
    puts "Roll back complete...."
  end
end
```

现在我们为 rake 准备好了 sandbox 容器了。以后我们可以这么执行 `bin/rake sandbox <name of your rake task>`

```bash
$ bin/rake sandbox employee_update_remove
```

## 原文链接

* [Sandboxed Rake Tasks for Your Rails app!](https://medium.com/@manoj_g33k/sandboxed-rake-tasks-for-your-rails-app-c6991b3fe1d5)
