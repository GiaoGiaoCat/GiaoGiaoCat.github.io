---
layout: post
title:  "Rails 的事务"
date:   2018-05-13 10:30:00
categories: rails
tags: tip
author: "Victor"
---

## 入门

### 为什么要使用事务

* 事务是指并发控制的单位，是用户定义的一个操作序列。
* 简单的来说事务里面的多个操作，要么都不执行，要么一起执行。
* 事务可以帮助开发者保证应用中的数据一致性。

常见的使用事务的场景是银行转账，钱从一个账户转移到另外一个账户。如果中间的某一步出错，那么整个过程应该重置 rollback。

```ruby
ActiveRecord::Base.transaction do  
  david.withdrawal(100)  
  mary.deposit(100)  
end
```

## 使用

### 在 Rails 中使用事务

Rails 中，通过 ActiveRecord 对象的类方法或者实例方法即可实现事务：

```ruby
Client.transaction do  
  @client.users.create!  
  @user.clients(true).first.destroy!  
  Product.first.destroy!  
end  

@client.transaction do  
  @client.users.create!  
  @user.clients(true).first.destroy!  
  Product.first.destroy!  
end
```

你不用关心是在 class 还是在 object 上调用的 `transaction` 反正最后都委托给了 [current thread's database connection](https://bibwild.wordpress.com/2014/07/17/activerecord-concurrency-in-rails4-avoid-leaked-connections/)。如果没有 ActiveRecord 的 class 的话，直接用 `ActiveRecord::Base.transaction do ... end` 也行。

```ruby
def copy_invoice(original)
  Invoice.transaction do
    duplicate = Invoice.create!(original.recipient)
    original.items.each do |item|
      duplicate.items.create!(article: item.article, amount: item.amount)
    end
  end
end
```

事务是和一个数据库连接绑定在一起的，而不是某个 model 对象，所以上面的例子中每个事务中均含有多个不同的 model 是完全可以的。**也只有在对多个纪录进行操作，并且希望这些操作作为一个整体的时候，事务才是必要的。**

### 处理异常

```ruby
def transfer_money(amount)
  ApplicationRecord.transaction do
    john.update!(money: john.money + amount)
    ted.update!(money: ted.money - amount)
  end

rescue ActiveRecord::RecordInvalid
  puts "Oops. We tried to do an invalid operation!"
end
```

### 触发回滚

事务通过 rollback 过程把记录的状态进行重置。在 Rails 中 rollback 只会被一个 `exception` 触发。这是非常关键的一点，很多事务块中的代码不会触发异常，因此即使出错，事务也不会回滚。

```ruby
# Wrong
ActiveRecord::Base.transaction do  
  david.update_attribute(:amount, david.amount -100)  
  mary.update_attribute(:amount, 100)  
end
```

因为 Rails 中，`#update_attribute` 方法在调用失败的时候也不会触发 `exception`，它只是简单的返回 `false` ，因此必须确保 transaction 块中的函数在失败时会抛异常。

```ruby
# Right
ActiveRecord::Base.transaction do  
  david.update_attribute!(:amount, david.amount -100)  
  mary.update_attribute!(:amount, 100)  
end
```

另一个常见的问题是 `find_by` 方法在找不到记录的时候回返回 `nil`，只有 `#find` 在找不到记录的时候才会触发异常 `ActiveRecord::RecordNotFound`。

```ruby
# Wrong
ActiveRecord::Base.transaction do  
  david = User.find_by_name("david")  
  if(david.id != john.id)  
    john.update_attributes!(:amount => -100)  
    mary.update_attributes!(:amount => 100)  
  end  
end
```

这里 `nil` 对象也有一个 id 方法，导致记录没有被找到的错误被隐藏，因此下面的代码被错误的执行了。这就意味着，有的时候在某些场景下，我们需要人工抛异常。

```ruby
# Right
ActiveRecord::Base.transaction do  
  david = User.find_by_name("david")  
  raise ActiveRecord::RecordNotFound if david.nil?  
  if(david.id != john.id)  
    john.update_attributes!(:amount => -100)  
    mary.update_attributes!(:amount => 100)  
  end  
end
```

**需要注意 `ActiveRecord::Rollback` 实际上并不会引发错误，它只是用于让事务本身进行回滚，而你也不需要在外部进行 catch 和处理它。**

```ruby
def transfer_money
  ApplicationRecord.transaction do
    john.update!(money: john.money + 100)
    ted.update!(money: ted.money - 100)
    raise ActiveRecord::Rollback if john.account_is_blocked?
  end
end
```

## 进阶

### 何时使用嵌套事务

下面这个例子中，每个事务会彼此独立进行回滚。但仍绑定到相同的数据库连接上，而我们又没有 rescue 任何错误，所以内部事务中的错误会回滚外部事务。

```ruby
def transfer_money
  ApplicationRecord.transaction do
    john.update!(money: john.money + 100)
    ted.update!(money: ted.money - 100)

    ApplicationRecord.transaction do
      transfer.create!(amount: 100, receiver: john, sender: ted)
    end
  end
end
```

日常开发中，经常会遇到错误使用或者过多使用嵌套的问题。当你把一个 `transaction` 嵌套在另外一个事务之中时，就会存在父事务和子事务，这种写法有时候会导致奇怪的结果。比如下面来自于 Rails API 文档的例子：

```ruby
User.transaction do  
  User.create(:username => 'Kotori')  
  User.transaction do  
    User.create(:username => 'Nemu')  
    raise ActiveRecord::Rollback  
  end  
end
```

由于 `ActiveRecord::Rollback` 不会传播到上层的方法中去，因此这个例子中，父事务并不会收到子事务抛出的异常。因为子事务块中的内容也被合并到了父事务中去，因此这个例子中，两条 User 记录都会被创建！

可以把嵌套事务这样理解，子事务中的内容被归并到了父事务中，这样子事务变空。

为了保证一个子事务的 rollback 被父事务知晓，必须手动在子事务中添加 `:require_new => true` 选项。比如下面的写法：

```ruby
User.transaction do  
  User.create(:username => 'Kotori')  
  User.transaction(:requires_new => true) do  
    User.create(:username => 'Nemu')  
    raise ActiveRecord::Rollback  
  end  
end
```

事务是跟当前的数据库连接绑定的，因此，如果你的应用同时向多个数据库进行写操作，那么必须把代码包裹在一个嵌套事务中去。比如：

```ruby
Client.transaction do  
  Product.transaction do  
    product.buy(@quantity)  
    client.update_attributes!(:sales_count => @sales_count + 1)  
  end  
end  
```

### 事务相关的回调

上面提到 `#save` 和 `#destroy` 方法被自动包裹在一个事务中，因此相关的回调，比如 `#after_save` 仍然属于事务的一部分，因此回调代码也有可能被回滚。

因此，如果你希望代码在事务外部执行的话，那么可以使用 `#after_commit` 或者 `#after_rollback` 这样的回调函数。

### 事务陷阱

不要在事务内部去捕捉 `ActiveRecord::RecordInvalid` 异常。因为某些数据库下，这个异常会导致事务失效，比如 Postgres。一旦事务失效，要想让代码正确工作，就必须从头重新执行事务。

另外，测试回滚或者事务回滚相关的回调时，最好关掉 `transactional_fixtures` 选项，一般的测试框架中，这个选项是打开的。

### 常见的事务反模式

* 单条记录操作时使用事务
* 不必要的使用嵌套式事务
* 事务中的代码不会导致回滚
* 在 controller 中使用事务

## 事务和锁的区别

这一部分请看相关中的 Differences between transactions and locking。

事务和锁都可以用来解决并发数据访问的问题，但是二者不能混为一谈。

所有 `callbacks` 和 `nested attributes processing` 都会自动包裹在一个事务中 [automatically run inside a transaction](https://makandracards.com/makandra/20301-cancel-the-activerecord-callback-chain)。

### 两个事务同时发生

比如上面的 `copy_invoice` 事务，是可以被两个线程同时调用的，并且不会抛出任何异常，就是正常的执行。

两个并发的事务，都会尝试在原子操作中提交它们的更改。成功不成功取决于我们代码的实现。比如，两个事务都尝试插入相同的 unique key，那就先提交的成功，后提交的抛错并回滚。。

另外就是如果两个事务是在两个 threads 中执行的，那么事务之间是无法获取另外一个 thread 中事务的状态的。

### 当心事务引发的死锁 deadlocks

在提交事务的时候，可能会触发死锁。

```ruby
def johnify!
  ApplicationRecord.transaction do
    User.find_each{ |user| user.update!(name: "John") }
  end
end
```

加入你有类似这样的事务，它会锁住许多行，就很容易造成死锁，详细的内容可以看参考连接中的 Prevent MySQL deadlocks in your Rails application。

```
ActiveRecord::StatementInvalid: Mysql::Error: Lock wait timeout exceeded; try restarting transaction
```

For instance, transaction A wants to change row #1 and row #2. Transaction B wants to change the same rows, but in different order (row #2, then row #1). Here is what happens:

* Transaction A locks row #1
* Transaction B locks row #2
* Transaction A tries to lock row #1, but can't
* Transaction B tries to lock row #2, but can't
* Both transactions wait forever
* MySQL times out after 50 seconds and raises an error in both database connections
* Both Rails worker processes raise ActiveRecord::StatementInvalid with a confusing error message

### 使用锁来防止并发访问

使用锁来确保代码只能在单一进程中执行。

```ruby
def transfer_money(source_id, target_id, amount)
  source_account = Account.find(source_id)
  target_account = Account.find(target_id)  
  source_account.balance -= amount
  target_account.balance += amount  
  source_account.save!
  target_account.save!
end
```

Let's say two users try to transfer 5 units of currency from the same source account to the same target account. Here is what can happen:

* Thread A retrieves both accounts. It sees that source_account and target_account both have a balance of 100.
* Thread B retrieves both accounts. It sees that source_account and target_account both have a balance of 100.
* Thread A sets source_account.balance to 95 and target_account.balance to 105.
* Thread A saves both accounts and terminates.
* Thread B also sets source_account.balance to 95 and increases target_account.balance to 105.
* Thread B saves both accounts and terminates.

What you should do instead is to wrap critical code in a lock. The lock ensures only a single thread may access it at a single time:

```ruby
def transfer_money(source_id, target_id, amount)
  Lock.acquire('money-transfer-lock') do
    source_account = Account.find(source_id)
    target_account = Account.find(target_id)  
    source_account.balance -= amount
    target_account.balance += amount  
    source_account.save!
    target_account.save!
  end
end
```

## 相关

* [Differences between transactions and locking](https://makandracards.com/makandra/31937-differences-between-transactions-and-locking)
* [Rails的事务和锁](https://blog.csdn.net/feigeswjtu/article/details/51830874)
* [Rails transactions: The Complete Guide](http://codeatmorning.com/rails-transactions-complete-guide/)
* [Prevent MySQL deadlocks in your Rails application](https://www.brightbox.com/blog/2014/11/13/preventing-mysql-deadlocks/)
* [Troubleshooting and avoiding deadlocks — MySQL/Rails](https://hackernoon.com/troubleshooting-and-avoiding-deadlocks-mysql-rails-766913f3cfbc)
* [Rails transactions and locks… a failed story](https://medium.com/dak42/rails-transactions-and-locks-a-failed-story-5c12e5b6e71d)
