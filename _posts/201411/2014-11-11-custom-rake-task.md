---
layout: post
title:  "Custom Rake Tasks"
date:   2014-11-11 10:30:00
categories: rails
tags: rake
author: "Victor"
---

## 创建 task

通常有 3 种方法创建自己的 rake task

1. 大神啥也不想，打开编辑器就开搞
2. 有点小聪明的人喜欢复制别人的代码
3. 屌屌的 Rails 程序员喜欢 ```rails g task my_namespace my_task1 my_task2```

上面的代码会生成 **lib/tasks/my_namespace.rake**

```ruby
namespace :my_namespace do
  desc "TODO"
  task :my_task1 => :environment do
  end

  desc "TODO"
  task :my_task2 => :environment do
  end
end
```

## rake 的一些知识

```ruby
task :text do
  puts "hello, world"
end

task :say => :text do
  puts "Yeah"
end
```

```bash
rake say

Hello, world
Yeah
```

通过上面的例子可以看出来，```:say => :text``` 的意思了吧。我们 ```:winner => :environment``` 的意思是，在执行该任务代码之前，先加载环境。

下面的代码演示了，如何利用 ```namespace, method```。

```ruby
namespace :pick do
  desc "Pick a random user as the winner"
  task :winner => :environment do
    puts "Winner: #{pick(User).name}"
  end

  desc "Pick a random product as the prize"
  task :prize => :environment do
    puts "Prize: #{pick(Product).name}"
  end

  desc "Pick a random prize and winner"
  task :all => [:prize, :winner]

  def pick(model_class)
    model_class.find(:first, :order => 'RAND()')
  end
end
```

## 相关链接

* <<Rake Task Management Essentials>>
* [Using the Rake Build Language](http://www.martinfowler.com/articles/rake.html)
* [Rake Tutorial](http://jasonseifer.com/2010/04/06/rake-tutoria)
* [How to Write Rock Solid Rake Tasks](http://bugroll.com/rock-solid-rake-tasks.html)
