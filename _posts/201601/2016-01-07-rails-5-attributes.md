---
layout: post
title:  "Introduction to Rails 5 Attributes"
date:   2016-01-07 08:00:00
categories: rails
tags: rails5
author: "Victor"
---

在 Rails 1.0 面世 10 周年之际，Rails 5 终于发布了。虽然该版本的主角毫无疑问是 `ActionCable`，但是也有一些首次引入的新功能很棒。

### Types of Changes

`ActiveRecord Attributes` 是特别值得介绍的新功能。它允许开发者给某个属性指定一个特定的类型和一个可选的默认值。它不是严格类型的验证，但它明确定义了提供类型强制转换，这一点非常有用。

假如我们有一个交易系统：

```ruby
class CreateTransactions < ActiveRecord::Migration
  def change
    create_table :transactions do |t|
      t.integer :user_id
      t.string :item_name
      t.integer :quantity
      t.string :success
      t.decimal :price
      t.timestamps(null: false)
    end
  end
end

class Transaction < ActiveRecord::Base
end
```

我们在实际开发中都遇到这样的情况，因为历史原因 **success** 字段是字符串而不是布尔型。虽然这个问题看似微不足道，一个系统可能有数亿的交易存在（想得美）。

你们穷到没有数据库专家，所以不能修改整列的类型，因为你不知道怎么样在系统不停机的情况下做到这一点。在这种情况下 `Attributes` 就可以用来救场了。

很简单，只需要添加如下一行到 `Transaction` 模型：

```ruby
class Transaction < ActiveRecord::Base
  attribute :success, :boolean
end
```

现在看起来，就像类型已经被强制转换过了一样：

```ruby
transaction = Transaction.new(success: 'yes')
transaction.success
# => true

transaction = Transaction.new(success: 'f')
transaction.success
# => false

transaction = Transaction.new(success: 0)
transaction.success
# => false
```

像你看到的一样，模式得到了改进。除了原生的 **SQL** 语句，代码现在可以防止 `transactions` 表的 `success` 出现像 `maybe` 一样的字符串了。

在 `Attributes` 出现之前，我们一般用 `callback` 或自定义的 `setter` 方法来解决这件事。现在，只用内置的强制类型转换方案就可以完美的解决这个问题了。

### The Friendly Type

通过查看 `ActiveRecord` 的源码，我们发现 `Attributes` 支持如下字段：

```ruby
#Supported Types

:big_integer
:binary
:boolean
:date
:date_time
:decimal
:float
:integer
:string
:text
:time
```

在系统中每种类型都有自己的用处和价值。它们存在的理由很简单：提高开发者和框架之间的交互体验。

`Attribute` 功能并不受数据库字段的限制。如果一个属性仅在对象的生命周期中使用，我们仍然可以使用强制类型转换。

例如：`confirmed_at` 属性对 `Transaction` 来说非常有用，但是它的格式经常改变。利用 `Attribute` 模块，我们可以让事情变得很简单。

```ruby
class Transaction
  attribute :confirmed_at, :date_time
end
```

我们不必在数据库中新增一个字段或者使用 `attr_accessor` 定义 `confirmed_at` 属性，仅用 `:date_time` 约束就完美的实现了这一功能：

```ruby
transaction = Transaction.new
transaction.confirmed_at = '2015-12-12 03:00'
transaction.confirmed_at
# => Sat, 12 Dec 2015 03:00:00 UTC +00:00

transaction = Transaction.new
transaction.confirmed_at = '2015/12/12'
transaction.confirmed_at
# => Sat, 12 Dec 2015 00:00:00 UTC +00:00
```

### Out of the Box Typing

除了刚刚提到的那些字段类型之外，其实 `Attributes` 还支持绑定对象类型。

为了帮助说明这个特点，我们可以假定数据库中的所有价格已更改为只处理美分，这有助于消除一些由浮点运算的复杂性。

```ruby
class MoneyType < ActiveRecord::Type::Integer
  def type_cast(value)
    if value.include?('$')
      price_in_dollars = value.gsub(/\$/, '').to_f
      price_in_dollars * 100
    else
      value.to_i
    end
  end
end
```

```ruby
class Transaction < ActiveRecord::Base
  attribute :price, MoneyType.new
end
```

```ruby
Transaction.where(price: '$10.00')
# => SELECT * FROM transactions WHERE price = 1000
```

可见它的行为非常有趣，在程序内部把字符串转换成了整数类型。

虽然 `Attributes` 并不是解决这类问题的唯一方法，但是仍然是一种值得我们收入工具箱的技巧。

### 原文

* [Introduction to Rails 5 Attributes](http://jakeyesbeck.com/2015/12/20/rails-5-attributes)
