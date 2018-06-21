---
layout: post
title:  "Rails 中的单表继承"
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

如果你要跨子类进行查询，那么毫无疑问要用 STI。

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

## 使用多态来替换 STI

```ruby
# Suppose the AccountHolder parent class is used
# to simply have an association with an account so
# both corporations and people can have accounts.

class Account < ActiveRecord::Base
  belongs_to :account_holder
end

class AccountHolder < ActiveRecord::Base
  has_one :account
end

class Corporation < AccountHolder
end

class Person < AccountHolder
end
```

```ruby
# Just use a polymorphic association instead

class Account < ActiveRecord::Base
  belongs_to :account_holdable, polymorphic: true
end

class Corporation < AccountHolder
  has_one :account, as: :account_holdable
end

class Person < AccountHolder
  has_one :account, as: :account_holdable
end
```

```ruby
# As such, accounts would need the polymorphic columns
# `account_holdable_type` and `account_holdable_id` which
# would describe the class as well as the primary key, respectively.

class CreateAccounts < ActiveRecord::Migration[5.0]
  def change
    create_table :accounts do |t|

      # ... various fields

      t.integer :account_holdable_id
      t.string  :account_holdable_type

      t.timestamps
    end
  end
end
```

## 相关阅读

* [Rails 的单表继承和多态关联](https://segmentfault.com/a/1190000010016287)
* [STI and Polymorphic Associations](http://railscasts.com/episodes/394-sti-and-polymorphic-associations)
