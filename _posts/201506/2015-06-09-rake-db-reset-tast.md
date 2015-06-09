---
layout: post
title:  "rake 清空和重建数据库"
date:   2015-06-09 12:30:00
categories: rails
tags: rake
author: "Victor"
---

```rake db:reset``` 是內建的 task，會執行一整套流程。包含 db:drop => db:create
=> db:schema:load => db:seed。

如果想要把项目中的一切资料清除还需要执行 `rake log:clear` 和 `rake tmp:clear`，如果懒得输入这些命令，我们可以创建一个新的 task。

```ruby
rails g task dev build rebuild
```

```ruby
namespace :dev do
  desc "Rebuild system"
  task :build => ["tmp:clear", "log:clear", "db:drop", "db:create", "db:migrate"]
  task :rebuild => [ "dev:build", "db:seed" ]
end
```

或者

```ruby
namespace :dev do
  desc "TODO"
  task build: :environment do
    Rake::Task["tmp:clear"].invoke
    Rake::Task["log:clear"].invoke
    Rake::Task["db:drop"].invoke
    Rake::Task["db:create"].invoke
    Rake::Task["db:migrate"].invoke
  end

  desc "TODO"
  task rebuild: :environment do
    Rake::Task["dev:build"].invoke
    Rake::Task["db:seed"].invoke
  end
end
```

