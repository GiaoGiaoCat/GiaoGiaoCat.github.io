---
layout: post
title:  "Rails 的锁机制"
date:   2016-08-09 10:30:00
categories: rails
tags: tip
author: "Victor"
---

## 并发

数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。

只要有资源的争用就少不了使用各种锁，包括关系数据库中使用的悲观锁和乐观锁，本文主要讲我们日常开发中很大可能会用到的两种锁。

### 常见的并发冲突

1. 丢失更新：一个事务的更新覆盖了其它事务的更新结果，就是所谓的更新丢失。
  * 例如：用户 A 把值从 6 改为 2，用户 B 把值从 2 改为 6，则用户 A 丢失了他的更新。
2. 脏读：当一个事务读取其它完成一半事务的记录时，就会发生脏读取。
  * 例如：用户 A, B 看到的值都是 6，用户B把值改为 2，用户A读到的值仍为 6。

```ruby
sku = User.find_by_id(sku_id)  
ActiveRecord::Base.transaction do  
  if(sku.stock > 0)  
    sku.update_attributes!(stock: sku.stock - 1)  
    Order.create!(order_attrs)  
  end              
end
```

在高并发的情况下，会出现这个问题，多个并发都同时执行到并满足第三行代码的条件，if 条件下的语句都会执行，就会出现超卖的问题。

```ruby
sku = User.find_by_id(sku_id)  
ActiveRecord::Base.transaction do  
  sku.lock!  
  if(sku.stock > 0)  
    sku.update_attributes!(stock: sku.stock - 1)  
    Order.create!(order_attrs)  
  end              
end
```

这样就不会出现超卖的情况了，多个并发同时执行并满足第四行代码时，if 条件下的语句也都会执行，lock 语句会执行: `select * from ... where ... for update`，查出的数据是最新的数据，所以不会出现这个问题。

## 悲观锁和乐观锁

### 什么是悲观锁 Pessimistic Locking

去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会 block 直到它拿到锁。悲观锁在事务开始之前就去尝试获得写权限，事务结束后释放锁。

* 在资源争用比较严重的时候比较合适
* 于同一行记录，只有一个写事务可以并行
* 传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁，因此，在整个数据处理过程中，将数据处于锁定状态
* 经常产生冲突，上层应用会不断的进行 retry，降低了性能，这种情况下用悲观锁就比较合适

一些常用的场景是电商系统中涉及到订单的部分，对于这种会对订单状态发生改变的操作，内部一般对这种操作都做加锁处理。

* 用户支付完成后可能会同时有多条支付成功的通知
* 订单改价的同时可能用户正在支付等等

悲观锁的实现，往往依靠数据库提供的锁机制。

一个典型的倚赖数据库的悲观锁调用 `select * from accounts where name=”Erica” for update` 这条 sql 语句锁定了 accounts 表中所有符合检索条件 `name="Erica"` 的记录。本次事务提交之前（事务提交时会释放事务过程中的锁），外界无法修改这些记录。

#### 在 Rails 的 ActiveRecord 使用悲观锁

```ruby
# select * from products where id=1 for update
Product.lock.find(1)
# 注意，这种最终会导致一个行锁

# select * from product where sku = '12343243' limit 1 for update
Product.where(sku: '12343243').lock(true).first
# 注意，这里可不是行锁，这里会是一个表锁
```

`select * from where xxx for update` 时，在 repeat read 的隔离级别下，MySQL 加锁机制取决于 sku 的索引。

* **如果 name 没有索引，则锁全表**
* 如果 name 有普通索引，则锁一个区间 - range lock
* 如果 name 是唯一索引，仅仅锁一行
* 如果 name 是主键，仅仅锁一行

如果查询的条件没有落在索引上，最好不要这样来用。Rails 提供了一个很方便的方法 `with_lock` 来锁住单个记录，并且内嵌在事务之中。下面代码中的两段是等价的：

```ruby
account = Account.find(1)
Account.transaction do
    account.lock!
    account.balance -= 100
    account.save!
end

# 和下面是等价的
account.with_lock do
    account.balance -= 100
    account.save!
end
```

### 什么是乐观锁 Optimistic Locking

在提交事务之前，大家可以各自修改数据，但是在提交事务的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。

* 适合在资源争用不激烈的时候使用，即冲突真的很少发生的时候
* 适用于多读的应用类型，省去了锁的开销，加大了系统的整个吞吐量
* 数据库如果提供类似于 `write_condition` 机制的其实都是提供的乐观锁

例如：也许会遇到两个人在接近的时间，同时去后台编辑一条记录的情况。可以通过 Rails 的乐观锁机制或者使用 updated_at 字段来判断用户要更新的对象，在数据库中的版本是否有变化。

乐观锁并不会使用数据库提供的锁机制。一般的实现乐观锁的方式就是记录数据版本。

#### Rails 内置的 `lock_version` 字段实现乐观锁

乐观锁本质上算是一个利用多版本管理来控制并发的技术。如果事务提交之后，数据库发现写入进程传入的版本号与目前数据库中的版本号不一致，说明有其他人已经修改过数据，不再允许本事务的提交。

使用乐观锁之前需要给数据库增加一列 `lock_version`，Rails 会自动识别这一列，向数据库提交数据的时候自动带上。

```ruby
p1 = Person.find(1)
p2 = Person.find(1)

p1.first_name = "Michael"
p1.save

p2.first_name = "should fail"
p2.save # Raises a ActiveRecord::StaleObjectError
```

你可以根据自己的需要添加重试机制。

```ruby
retry_times = 3

begin
    @order.with_lock do
        @order.set_paid!
    end
rescue ActiveRecord::StaleObjectError => e
    retry_times -= 1
    if retry_times > 0
        retry
    else
        raise e
    end
rescue => e
    raise e
end
```

注意如果是前端操作频繁，那么还需要把 `lock_version` 写入到 form 表单中，否则起不到锁的作用。

```ruby
# /db/migrations/20120820000000_add_lock_version_to_products.rb
class AddLockVersionToProducts < ActiveRecord::Migration
  def change
    add_column :products, :lock_version, :integer, default: 0, null: false
  end
end
```

```erb
<%= f.hidden_field :lock_version %>
```

```ruby
# /app/models/product.rb
def update_with_conflict_validation(*args)
  update_attributes(*args)
rescue ActiveRecord::StaleObjectError
  self.lock_version = lock_version_was
  errors.add :base, "This record changed while you were editing it."
  changes.except("updated_at").each do |name, values|
    errors.add name, "was #{values.first}"
  end
  false
end

# /app/controllers/products_controller.rb
def update
  @product = Product.find(params[:id])
  if @product.update_with_conflict_validation(params[:product])
    redirect_to @product, notice: "Updated product."
  else
    render :edit
  end
end
```

#### 使用 `updated_at` 来实现乐观锁

```erb
<%= f.hidden_field :original_updated_at %>
```

```ruby
class Product < ActiveRecord::Base
  belongs_to :category

  attr_writer :original_updated_at
  validate :handle_conflict, only: :update

  def original_updated_at
    @original_updated_at || updated_at.to_f
  end

  def handle_conflict
    if @conflict || updated_at.to_f > original_updated_at.to_f
      @conflict = true
      @original_updated_at = nil
      errors.add :base, "This record changed while you were editing. Take these changes into account and submit it again."
      changes.each do |attribute, values|
        errors.add attribute, "was #{values.first}"
      end
    end
  end
end
```

## 需要注意的地方

1. 一般，使用锁的时候和事务同时使用，所以 with_lock 是用的比较多的，而且尽量使用行锁而不是表锁。
2. 另外，也注意异常的处理，需要使用那些会抛异常的方法。

## 相关阅读

* [What is Optimistic Locking](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html)
* [What is Pessimistic Locking](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Pessimistic.html)
* [A GUIDE TO OPTIMISTIC LOCKING](https://www.engineyard.com/blog/a-guide-to-optimistic-locking)
* [Optimistic Locking](http://railscasts.com/episodes/59-optimistic-locking-revised)
* [Rails 中乐观锁与悲观锁的使用](https://ruby-china.org/topics/28963)
* [Rails 中的悲观锁和乐观锁](https://segmentfault.com/a/1190000009920888)
* [Understanding Locking in Rails ActiveRecord](http://thelazylog.com/understanding-locking-in-rails-activerecord/)
