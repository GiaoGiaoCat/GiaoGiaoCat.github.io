---
layout: post
title:  "Rails 中的代表继承"
date:   2018-05-31 12:00:00

categories: rails
tags: advanced
author: "Victor"
---

## 单表继承

### 什么是单表继承

Single Table Inheritance (separate classes, one table) 单表继承就是一张表, 分多个类使用。

### 何时使用单表继承

当各个模型之间只有细微的差别时，在使用前首先要问自己几个问题：

1. 是所有的对象都继承自一个类吗？
2. 是否需要对数据库中的所有对象进行查询操作？
3. 是所有对象都有相同的属性，但是不同的行为吗？

### 具体实现

```bash
rails g migration add_type_to_users type
rails g migration remove_guest_from_users guest:boolean
rake db:migrate
```

```ruby
class User < ActiveRecord::Base
  has_many :tasks, dependent: :destroy

  def guest?
    raise NotImplementError, "Must be implemented in subclasses."
  end  

  #etc...
end
```

```ruby
class Guest < User
  def guest?
    true
  end

  def move_to(user)
    tasks.update_all(user_id: user.id)
  end

  def name
    "Guest"
  end

  def task_limit
    10
  end

  def can_share_task?(task)
    false
  end

  def send_password_reset
  end
end
```

```ruby
class Member < User
  attr_accessor :username, :email, :password, :password_confirmation

  validates_presence_of :username, :email
  validates_uniqueness_of :username, allow_blank: true

  has_secure_password

  def guest?
    false
  end

  def name
    username
  end

  def task_limit
    1000
  end

  def can_share_task?(task)
    task.user_id == id
  end

  def send_password_reset
    UserMailer.password_reset(self).deliver
  end
end
```

## 抽象类 abstract_class

当你有几个子类要继承同一个父类，但是并不是 STI 的时候，在父类声明 `abstract_class`，在子类声明各自的 table_name。

```ruby
class Shape < ActiveRecord::Base
  self.abstract_class = true
end
```

```bash
Polygon = Class.new(Shape)
Square = Class.new(Polygon)

Shape.table_name   # => nil
Polygon.table_name # => "polygons"
Square.table_name  # => "polygons"
Shape.create!      # => NotImplementedError: Shape is an abstract class and cannot be instantiated.
Polygon.create!    # => #<Polygon id: 1, type: nil>
Square.create!     # => #<Square id: 2, type: "Square">
```

## 相关阅读

* [Rails 的单表继承和多态关联](https://segmentfault.com/a/1190000010016287)
* [STI and Polymorphic Associations](http://railscasts.com/episodes/394-sti-and-polymorphic-associations)
