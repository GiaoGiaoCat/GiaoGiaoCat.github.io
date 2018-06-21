---
layout: post
title:  "Rails 中的多表继承"
date:   2018-06-20 12:00:00

categories: rails
tags: advanced
author: "Victor"
---

## 多表继承

### 什么是多表继承

Multiple Table Inheritance 多表继承就是各个子类有单独的表，但是通过父类来共享通用的方法。

### 何时使用多表继承

1. 子类之间的 Fields/Columns 差异极大，但是需要共享类似的行为
2. 多表继承可以一定程度上解决单表过大的问题（对此我个人持反对意见，当 MTI 分表之后造成单表体积巨大，通过分片 sharding 来解决表的体积时，MTI 让你将来的工作更麻烦）

### 具体实现

```ruby
class Account < ActiveRecord::Base
  def withdraw(amount)
    # ...
  end

  def deposit(amount)
    # ...
  end

  def close!
    update_attribute!(closed_at, Time.current)
  end
end

# These three types of accounts inherit from a common base class
# so they can all share a common `#close` method.  However, each
# class has its own class-specific methods as well as their own
# tables.

class CorporateAccount < Account
  set_table_name "corporate_accounts"
  def close!
    inform_share_holders!
    super
  end
end

class SmallBusinessAccount < Account
  set_table_name "small_business_accounts"
  def close!
    inform_mom_and_pop!
    super
  end
end

class PersonalAccount < Account
  set_table_name "personal_accounts"
  def close!
    inform_dude_or_dudette!
    super
  end
end
```

```ruby
# As such, we need to create three tables. A very un-DRY approach.

class CreateCorporateAccounts < ActiveRecord::Migration
  def change
    create_table :corporate_accounts do |t|
      t.float     :balance
      t.string    :tax_identifier
      t.date_time :closed_at
      # ... more column fields #
      t.timestamps
    end
  end
end

class CreateSBAccounts < ActiveRecord::Migration
  def change
    create_table :small_business_accounts do |t|
      t.float     :balance
      t.string    :tax_identifier
      t.date_time :closed_at
      # ... more column fields #
      t.timestamps
    end
  end
end

class CreatePersonalAccounts < ActiveRecord::Migration
  def change
    create_table :personal_accounts do |t|
      t.float     :balance
      t.string    :tax_identifier
      t.date_time :closed_at
      # ... more column fields #
      t.timestamps
    end
  end
end
```

## 抽象类 abstract_class

当你有几个子类要继承同一个父类，但是父类并不存在对应的 table，可以在父类声明 `abstract_class`，在子类声明各自的 table_name。

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

* [When To Use Single Table Inheritance vs Multiple Table Inheritance](https://medium.com/@User3141592/when-to-use-single-table-inheritance-vs-multiple-table-inheritance-db7e9733ae2e)
* [Multiple table inheritance with ActiveRecord](http://hakunin.com/mti)
