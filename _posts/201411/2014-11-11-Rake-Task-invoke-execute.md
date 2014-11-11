---
layout: post
title:  "Rake Task 中 invoke 跟 execute 的不同"
date:   2014-11-11 11:10:00
categories: rails
tags: rake
author: "Victor"
---

我们需要在一个 Task 里边调用另一个 Task 的时候，我们可以使用 ```Rake::Task['task_name'].invoke ``` 的方式。

但需要注意 ```Rake::Task#invoke``` 在默认情况下在整个运行过程中将只会被调用一次而已。

```ruby
# lib/tasks/demo.rake
namespace :demo do
  desc "Print 'Hello' string"
  task :say_hello do
    puts "Hello, World!"
  end
end
```

```bash
$ rake demo:say_hello
=> Hello, World!
```

```ruby
namespace :demo do
  # ....

  desc "Print 'Hello, World!' five times"
  task :say_five_hello do
    5.times do
      Rake::Task['demo:say_hello'].invoke
    end
  end
end
```

```bash
$ rake demo:say_five_hello
=> Hello, World!
```

结果就是，** Hello, World! ** 只打印了一次，也就是说，我们的 ```Rake::Task['demo:say_hello']``` 只被运行了一次。

我们有两种修改方案。第一种就是将上述代码进行修改：

```ruby
namespace :demo do
  # ...

  desc "Print 'Hello, World!' five times"
  task :say_five_hello do
    5.times do
      Rake::Task['demo:say_hello'].execute
    end
  end
end
```

或者

```ruby
namespace :demo do
  # ...

  desc "Print 'Hello, World!' five times"
  task :say_five_hello do
    5.times do
      Rake::Task['demo:say_hello'].reenable
      Rake::Task['demo:say_hello'].invoke
    end
  end
end
```

## 原文出处

* [注意Rake Task中invoke方法跟execute方法的不同](http://martin91.github.io/blog/2014/03/21/zhu-yi-rake-taskzhong-invokegen-executefang-fa-de-bu-tong/)
