---
layout: post
title:  "Build Awesome Command-Line Applications in Ruby: 01 Have a Clear and Concise Purpose"
date:   2014-09-25 18:20:00
categories: ruby
tags: terminal
author: "Victor"
---

## Problem 1: Backing Up Data

假设我们公司有个小团队要开发自己新的核心产品。这是一款拥有大量数据资料和高度复杂，并且有很多新特性和功能的 Web 应用。为了实现它，我们决定采用敏捷开发的方法，打算使用两周一次的 sprints。在每个 sprint 中，我们需要一些 user stories 代表我们正在做的工作列表。我们需要在共享的开发环境中，完成每个 user story ，已代表我们的任务正常完成。

为了验证我们的功能特点，我们将准备一组精心挑选的数据，模拟所有可能遇到的边界情况和用户流程。设置这些数据是很耗费时间的，因为我们程序非常复杂。所以，尽管这些数据是假的，我们也要像对待真正的生产数据一样对它进行备份。根据我们的工作进度，我们要不停的修改数据。我们希望在当前迭代中的每一天来区分数据库的状态。并且在每一个迭代之后来备份数据库。例如，我们在第3次迭代的第5天可以访问迭代 1 和迭代 2 的数据，也可以访问第 3 次迭代中第 3 天的数据。

和大多数团队一样，在我们公司我们不能依靠系统管理员进行备份。因为我们是一个刚刚起步的团队，啥都只能靠自己。一个 command-line 工具就能解决如下事情：

* 完整的 dump 整个 mysql 数据库
* 根据备份的日子命名备份文件
* 在最终迭代的时候，使用不同的命名
* 压缩备份文件
* 当迭代完成的时候可以删除备份

下面开始搞起。我们建立一个 Hash 包含所有需要备份的数据库信息，遍历它，然后使用 Ruby 的反引号操作符来调用 mysqldump ，然后再使用 gzip1。我们也将检查第一个参数：如果它存在，则进行 "end-of-iteration" 备份。下面是我们最初实现的代码：

```ruby
# have_a_purpose/db_backup/bin/db_backup_initial.rb

#!/usr/bin/env ruby
databases = {
  big_client: {
    database: 'big_client',
    username: 'big',
    password: 'big',
  },
  small_client: {
    database: 'small_client',
    username: 'small',
    password: 'p@ssWord!',
  }
}

end_of_iter = ARGV.shift

databases.each do |name,config|
  if end_of_iter.nil?
    backup_file = config[:database] + '_' + Time.now.strftime('%Y%m%d')
  else
    backup_file = config[:database] + '_' + end_of_iter
  end
  mysqldump = "mysqldump -u#{config[:username]} -p#{config[:password]} " + "#{config[:database]}"

  `#{mysqldump} > #{backup_file}.sql`
  `gzip #{backup_file}.sql`
end
```

请注意上面是如何使用 ARGV 的，它是一个数组，Ruby 把我们命令后面跟随的参数传递给它。在本例中：

```ruby
$ db_backup_initial.rb
# => creates big_client_20110103.sql.gz
# => creates small_client_20110103.sql.gz
$ db_backup_initial.rb iteration_3
# => creates big_client_iteration_3.sql.gz
# => creates small_client_iteration_3.sql.gz
```

这个应用程序有很多的问题和大量的改进余地。这本书的其余部分将解决这些问题，但我们现在要解决的最大一个问题：这个应用程序没有一个清晰，简明的目的。我们想象一个可能的场景：添加第三个数据库进行备份。

为了支持这一点，我们需要编辑代码，修改数据库 Hash，并重新部署应用程序到数据库服务器。为了让这个应用程序简单。我们决定让它每次只备份一个数据库，当有 3 个数据库的时候，只需要重复执行 3 次就行。这样以后就方便多了。

我们把数据库名称，用户名，密码都通过参数的方式传递给命令行：

```ruby
#!/usr/bin/env ruby
database = ARGV.shift
username = ARGV.shift
password = ARGV.shift
end_of_iter = ARGV.shift
if end_of_iter.nil?
  backup_file = database + Time.now.strftime("%Y%m%d")
else
  backup_file = database + end_of_iter
end
`mysqldump -u#{username} -p#{password} #{database} > #{backup_file}.sql`
`gzip #{backup_file}.sql`
```

现在我们可以这么调用该命令行：

```bash
$ db_backup.rb big_client big big
# => creates big_client_20110103.sql.gz
$ db_backup.rb small_client small "p@ssWord!"
# => creates small_client_20110103.sql.gz
$ db_backup.rb big_client big big iteration_3
# => creates big_client_iteration_3.sql.gz
$ db_backup.rb medium_client medium "med_pass" iteration_4 # => creates medium_client_iteration_4.sql.gz
```

不管怎么说，我们的确让应用程序的维护更简单了。为了保证每天定时运行，可能会使用到 ```cron``` 之类的工具。本书的其余章节会继续改进这个脚本。

### Shebang符号(#!)

上面的 ```db_backup_initial.rb``` 我们用到了 ```Shebang```，需要注意的是因为 ```db_backup_initial.rb``` 并不在我们的 ```$PATH``` 中，所以我们要执行它，一定要用决定路径或者相对路径的方式来执行。

```bash
$ /home/you/project/have_a_purpose/db_backup/bin/db_backup_initial.rb
or
$ ./db_backup_initial.rb
```

* [Linux上的Shebang符号(#!)](http://smilejay.com/2012/03/linux_shebang/)
* [Ruby Programming/Hello world](http://en.wikibooks.org/wiki/Ruby_Programming/Hello_world)

## Problem 2: Managing Tasks

多数软件开发组织使用某种任务管理和故障单系统。像 JIRA，Bugzilla，Pivotal Tracker 这些工具提供了丰富的功能，用于管理最复杂的工作流程和任务。编程时的一个常见方法是取大功能，分解成小的任务。假设我们要为公司的 Web App 添加服务条款页面，需要修改帐号注册页面，要求用户接受服务的新条款。

在我们司的任务管理工具，我们可能会看到类似的任务 "Add Terms of Service Checkbox to Signup Page." 老板和项目经理当然会只关心这个任务是否完成，但是作为程序员我们会把这个功能拆分成任务列表：

* 为数据库添加 “accepted terms on date.” 字段
* 跪求 DBA 允许添加这个字段
* 在 Web 表单中增加 checkbox
* 添加业务逻辑，确保当注册的时候这个 box 是被勾选的
* 代码审查和测试确保功能正确

由于在我们的任务管理系统中做这种细颗粒度的任务追踪实在是太麻烦了，我们可以用纸和笔卸下来。或者用一个简单的工具来完成。

为了让逻辑清晰，我们创建 3 个 command-line 应用，每个用来处理不同的事情。**todo-new.rb** 用来添加新的任务。**todo-list.rb** 用来列表当前的任务。**todo-done.rb** 用来标记完成一个任务。我们会把所有的任务存在当前目录下的 **todo.txt** 文件中。

```bash
$ todo-new.rb "Add new field to database for 'accepted terms on date'"
Task added
$ todo-new.rb "Get DBA approval for new field."
Task added
$ todo-list.rb
1 - Add new field to database for 'accepted terms on date'
Created: 2011-06-03 13:45
2 - Get DBA approval for new field. Created: 2011-06-03 13:46
$ todo-done.rb 1
Task 1 completed
$ todo-list.rb
1 - Add new field to database for 'accepted terms on date'
    Created:   2011-06-03 13:45
    Completed: 2011-06-03 14:00
2 - Get DBA approval for new field.
    Created:   2011-06-03 13:46
```

**todo-new.rb** 从命令行中读取 task 然后追加到 **todo.txt** 中，并且要有 timestamp。

```ruby
#have_a_purpose/todo/bin/todo-new.rb

#!/usr/bin/env ruby
new_task = ARGV.shift
File.open('todo.txt','a') do |file|
  file.puts "#{new_task},#{Time.now}"
  puts "Task added."
end
```

**todo-list.rb** 从 **todo.txt** 读取任务，然后生成 ID。

```ruby
#have_a_purpose/todo/bin/todo-list.rb

#!/usr/bin/env ruby
File.open('todo.txt','r') do |file|
  counter = 1
  file.readlines.each do |line|
    name, created, completed = line.chomp.split(/,/)
    printf("%3d - %s\n", counter, name)
    printf(" Created : %s\n", created)
    unless completed.nil?
      printf(" Completed : %s\n", completed)
    end
    counter += 1
  end
end
```

**todo-done.rb** 从 **todo.txt** 读取任务，并写入文件。当我们找到需要完成的任务，进行操作并停止执行。

```ruby
#have_a_purpose/todo/bin/todo-done.rb

#!/usr/bin/env ruby
task_number = ARGV.shift.to_i

File.open('todo.txt','r') do |file|
  File.open('todo.txt.new','w') do |new_file|
    counter = 1
    file.readlines.each do |line|
      name, created, completed = line.chomp.split(/,/)
      if task_number == counter
        new_file.puts("#{name},#{created},#{Time.now}")
        puts "Task #{counter} completed"
      else
        new_file.puts("#{name},#{created},#{completed}")
      end
      counter += 1
    end
  end
end
`mv todo.txt.new todo.txt`
```

我们这个应用有个大问题：一旦我们要添加一个新的字段，就得修改3个文件。真够傻的。所以我决定把这些命令合成一个文件，根据接收的不同参数来执行不同的功能。

```bash
$ todo new "Add new field to database for 'accepted terms on date'"
Task added
$ todo new "Get DBA approval for new field."
Task added
$ todo list
1 - Add new field to database for 'accepted terms on date'
Created: 2011-06-03 13:45
2 - Get DBA approval for new field. Created: 2011-06-03 13:46
$ todo done 1
Task 1 completed
$ todo list
1 - Add new field to database for 'accepted terms on date'
    Created:   2011-06-03 13:45
    Completed: 2011-06-03 14:00
2 - Get DBA approval for new field.
    Created:   2011-06-03 13:46
```

```ruby
#!/usr/bin/env ruby
TODO_FILE = 'todo.txt'

def read_todo(line)
  line.chomp.split(/,/)
end

def write_todo(file, name, created = Time.now, completed = '')
  file.puts("#{name},#{created},#{completed}")
end

command = ARGV.shift
case command
when 'new'
  new_task = ARGV.shift
  File.open(TODO_FILE,'a') do |file|
    write_todo(file,new_task)
    puts "Task added."
  end
when 'list'
  File.open(TODO_FILE,'r') do |file|
    counter = 1
    file.readlines.each do |line|
      name,created,completed = read_todo(line)
      printf("%3d - %s\n",counter,name)
      printf("  Created : %s\n",created)
      unless completed.nil?
        printf("  Completed : %s\n",completed)
      end
      counter += 1
    end
  end
when 'done'
  task_number = ARGV.shift.to_i
  File.open(TODO_FILE,'r') do |file|
    File.open("#{TODO_FILE}.new",'w') do |new_file|
      counter = 1
      file.readlines.each do |line|
        name,created,completed = read_todo(line)
        if task_number == counter
          write_todo(new_file,name,created,Time.now)
          puts "Task #{counter} completed"
        else
          write_todo(new_file,name,created,completed)
        end
        counter += 1
      end
    end
  end
  `mv #{TODO_FILE}.new #{TODO_FILE}`
end
```

## What Makes an Awesome Command-Line App

好的 command-line 应用具有如下特点：

* Easy to use - 使用简单
* Helpful - 提供帮助信息
* Plays well with others - 可以和其它应用交互
* Has sensible defaults but is configurable - 默认配置很好但是也可以自定义配置
* Installs painlessly - 安装简单
* Fails gracefully - 可以处理意外情况
* Gets new features and bug fixes easily - 可以方便的扩展和修复错误
* Delights users - 如果能输出彩色的效果更棒了

## Moving On

虽然上面的例子没啥了不起的，但是它向我们演示了如何使用 Ruby 来完成一个 command-line 应用帮助我们改善工作流程。本书的其他例子将会从这个基础上不断的完善，最终打造出一个核武器。

另外别忘了最重要的一点，你的工具一定要有一个清晰的目标：为了完成什么功能。
