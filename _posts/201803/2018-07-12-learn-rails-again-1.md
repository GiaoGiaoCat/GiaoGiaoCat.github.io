---
layout: post
title:  "重读 Rails Guide 1"
date:   2018-07-12 13:00:00

categories: rails
tags: learningnote
author: "Victor"
---

## Rails 入门
Rails 有自己的设计原则，认为问题总有最好的解决方法，并且有意识地通过设计来鼓励用户使用最好的解决方法，而不是其他替代方案。掌握了 **Rails 之道**，就可能获得生产力的巨大提升。否则，你的开发体验可能就不那么愉快。

Rails 哲学包含两大指导思想：

1. **不要自我重复（DRY）** DRY 是软件开发中的一个原则，意思是“系统中的每个功能都要具有单一、准确、可信的实现。”。不重复表述同一件事，写出的代码才更易维护、更具扩展性，也更不容易出问题。
2. **多约定，少配置** Rails 为 Web 应用的大多数需求都提供了最好的解决方法，并且默认使用这些约定，而不是在长长的配置文件中设置每个细节。

技巧：

1. 善用 `inspect` 方法，Rails 为很多类都扩展了 `inspect` 方法，它在调试的时候经常用到。
2. `form_for` 和 `link_to` 都可以在参数中以数组的方式为 URL 赋值。

```ruby
render plain: params[:article].inspect
```

```erb
<%= form_for([@article, @article.comments.build]) do |f| %>
<%= link_to 'Destroy Comment', [comment.article, comment], method: :delete, data: { confirm: 'Are you sure?' } %>
```

## 模型
### Active Record 基础
Active Record 负责处理数据和业务逻辑，创建和使用需要持久存入数据库中的数据。Active Record 实现了 Active Record 模式，是一种对象关系映射系统（Object Relational Mapping，ORM）。

Active Record 提供了很多功能，其中最重要的几个如下：

* 表示模型和其中的数据
* 表示模型之间的关系
* 通过相关联的模型表示继承层次结构
* 持久存入数据库之前，验证模型
* 以面向对象的方式处理数据库操作

Active Record 中的 “多约定少配置” 原则：

* 外键：使用 `singularized_table_name_id` 形式命名，例如 item_id，order_id。创建模型关联后，Active Record 会查找这个字段
* 主键：默认情况下，Active Record 使用整数字段 id 作为表的主键。使用 Active Record 迁移创建数据库表时，会自动创建这个字段
* created_at：创建记录时，自动设为当前的日期和时间
* updated_at：更新记录时，自动设为当前的日期和时间
* lock_version：在模型中添加乐观锁
* type：让模型使用单表继承
* (association_name)_type：存储多态关联的类型
* (table_name)_count：缓存所关联对象的数量。比如说，一个 Article 有多个 Comment，那么 comments_count 列存储各篇文章现有的评论数量

覆盖命名约定：

```ruby
# 使用 ActiveRecord::Base.table_name= 方法可以指定要使用的表名：
class Product < ApplicationRecord
  self.table_name = "my_products"
end

# 如果这么做，还要调用 set_fixture_class 方法，手动指定固件（my_products.yml）的类名：
class ProductTest < ActiveSupport::TestCase
  set_fixture_class my_products: Product
  fixtures :my_products
  ...
end

# 还可以使用 ActiveRecord::Base.primary_key= 方法指定表的主键：
class Product < ApplicationRecord
  self.primary_key = "product_id"
end
```

### Active Record 迁移
迁移是 Active Record 的一个特性，是以一致和轻松的方式按时间顺序修改数据库模式的实用方法。它使用 Ruby DSL，因此不必手动编写 SQL，从而实现了数据库无关的数据库模式的创建和修改。

如果想在迁移中完成一些 Active Record 不知如何撤销的操作，可以使用 reversible 方法，或者用 up 和 down 方法来代替 change 方法：

```ruby
class ChangeProductsPrice < ActiveRecord::Migration[5.0]
  def change
    reversible do |dir|
      change_table :products do |t|
        dir.up   { t.change :price, :string }
        dir.down { t.change :price, :integer }
      end
    end
  end
end
```

如果迁移名称是 AddXXXToYYY 或 RemoveXXXFromYYY 的形式，并且后面跟着字段名和类型列表，那么会生成包含合适的 add_column 或 remove_column 语句的迁移。

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string:index
$ bin/rails generate migration RemovePartNumberFromProducts part_number:string
$ bin/rails generate migration AddDetailsToProducts part_number:string price:decimal
$ bin/rails generate migration CreateProducts name:string part_number:string
$ bin/rails generate migration AddUserRefToProducts user:references
$ bin/rails g migration CreateJoinTableCustomerProduct customer product
```

`change_column` 是无法撤销的。在大多数情况下，Active Record 知道如何自动撤销用 `change` 方法编写的迁移。在 `change` 方法中能使用的方法在相关 Guide 中已有阐述，这里不复制了。

Active Record 在模型而不是数据库中声明关联，在模型中强制数据完整性。因此，像触发器、约束这些依赖数据库的特性没有被大量使用。

#### 数据库模式转储
Active Record 通过检查数据库生成的 db/schema.rb 文件或 SQL 文件才是数据库模式的可信来源。这两个可信来源不应该被修改，它们仅用于表示数据库的当前状态。

数据库模式转储有两种方式，可以通过 config/application.rb 文件的 config.active_record.schema_format 选项来设置想要采用的方式，即 :sql 或 :ruby。

`:ruby` 模式是数据库无关的，但不能表达数据库的特定项目，如触发器、存储过程或检查约束。在把数据库模式转储到 `db/structure.sql` 文件时，使用数据库特有的工具（通过执行 `db:structure:dump` 任务）。

**强烈建议将其纳入源码版本控制。**

## Active Record 数据验证

爆炸方法（例如 save!）会在验证失败后抛出异常。验证失败后，非爆炸方法不会抛出异常，`save` 和 `update` 返回 `false`，`create` 返回对象本身。

下列方法会跳过验证，不管验证是否通过都会把对象存入数据库，使用时要特别留意。

`decrement, decrement_counter, increment!, increment_counter, toggle!, touch, update_all, update_attribute, update_column, update_columns, update_counters` 使用 save 时如果传入 `validate: false` 参数，也会跳过验证。

### 数据验证辅助方法

* `acceptance` 检查表单提交时，用户界面中的复选框是否被选中，默认为 ['1', true]
* `validates_associated` 如果模型和其他模型有关联，而且关联的模型也要验证，要使用这个辅助方法。保存对象时，会在相关联的每个对象上调用 valid? 方法
* `confirmation` 检查两个文本字段的值是否完全相同，这个验证创建一个虚拟属性，其名字为要验证的属性名后加 _confirmation
* `exclusion` 检查属性的值是否不在指定的集合中
* `format` 属性的值是否匹配 :with 选项指定的正则表达式，使用 :without 选项，指定属性的值不能匹配正则表达式。
* `inclusion` 属性的值是否在指定的集合中
* `length` 验证属性值的长度
* `numericality` 检查属性的值是否只包含数字，默认情况下，匹配的值是可选的正负符号后加整数或浮点数
* `presence` 检查指定的属性是否为非空值，会在关联的对象上调用 `blank?` 和 `marked_for_destruction?` 方法。
* `absence` 验证指定的属性值是否为空，会在关联的对象上调用 `present?` 和 `marked_for_destruction?` 方法
* `uniqueness` 验证属性值是否是唯一的，**需要配合数据库的唯一性索引来确保不会出现相同的字段值**

```ruby
validates :terms_of_service, acceptance: { accept: 'yes' }
validates :eula, acceptance: { accept: ['TRUE', 'accepted'] }

has_many :books
validates_associated :books # 不要在关联的两端都使用 validates_associated，这样会变成无限循环。

validates :email, confirmation: true
# 只有 email_confirmation 的值不是 nil 时才会检查。所以要为确认属性加上存在性验证
# 还可以使用 :case_sensitive 选项指定确认时是否区分大小写。这个选项的默认值是 true
validates :email_confirmation, presence: true

validates :subdomain, exclusion: { in: %w(www us ca jp) }
validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/, message: "only allows letters" }
validates :registration_number, length: { minimum: 2, maximum: 500, in: 6..20, is: 6 }
validates :games_played, numericality: { only_integer: true } # numericality 默认不接受 nil 值
validates :name, :login, :email, absence: true
```

```ruby
class LineItem < ApplicationRecord
  belongs_to :order
  validates :order, presence: true # 如果要确保关联对象存在，需要测试关联的对象本身是否存在，而不是用来映射关联的外键
end

class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order # 为了能验证关联的对象是否存在，要在关联中指定 :inverse_of 选项
end
```

```ruby
class LineItem < ApplicationRecord
  belongs_to :order
  validates :order, absence: true # 要测试关联的对象本身是否为空，而不是用来映射关联的外键
end

class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order # 为了能验证关联的对象是否为空，要在关联中指定 :inverse_of 选项。
end
```

因为 `false.blank?` 的返回值是 true，所以如果要验证布尔值字段是否存在，要使用下述验证中的一个：

```ruby
validates :boolean_field_name, inclusion: { in: [true, false] }
validates :boolean_field_name, exclusion: { in: [nil] }
```

### validates_with 和 validates_each

* `validates_with` 把记录交给其他类做验证，参数是一个类或一组类，没有默认的错误消息。在做验证的类中要手动把错误添加到记录的错误集合中。
* 做验证的类在整个应用的生命周期内只会初始化一次，而不是每次验证时都初始化，所以使用实例变量时要特别小心。如果做验证的类很复杂，必须要用实例变量，可以用纯粹的 Ruby 对象代替。
* `validates_each` 使用代码块中的代码验证属性。

```ruby
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if options[:fields].any?{|field| record.send(field) == "Evil" }
      record.errors[:base] << "This person is evil"
    end
  end
end

class Person < ApplicationRecord
  validates_with GoodnessValidator, fields: [:first_name, :last_name]
end
```

```ruby
class Person < ApplicationRecord
  validate do |person|
    GoodnessValidator.new(person).validate
  end
end

class GoodnessValidator
  def initialize(person)
    @person = person
  end

  def validate
    if some_complex_condition_involving_ivars_and_private_methods?
      @person.errors[:base] << "This person is evil"
    end
  end
  # ...
end
```

```ruby
class Person < ApplicationRecord
  validates_each :name, :surname do |record, attr, value|
    record.errors.add(attr, 'must start with upper case') if value =~ /\A[[:lower:]]/
  end
end
```

### 常用的验证选项

`:message` 选项的值是一个字符串或一个 Proc 对象。

* 字符串消息中可以包含 `%{value}、%{attribute} 和 %{model}`
* Proc 形式的消息有两个参数：验证的对象，以及包含 `:model、:attribute 和 :value` 键值对的散列

```ruby

class Person < ApplicationRecord
  # 直接写消息
  validates :name, presence: { message: "must be given please" }

  # 带有动态属性值的消息。%{value} 会被替换成属性的值
  # 此外还可以使用 %{attribute} 和 %{model}
  validates :age, numericality: { message: "%{value} seems wrong" }

  # Proc
  validates :username,
    uniqueness: {
      # object = 要验证的 person 对象
      # data = { model: "Person", attribute: "Username", value: <username> }
      message: ->(object, data) do
        "Hey #{object.name}!, #{data[:value]} is taken already! Try again #{Time.zone.tomorrow}"
      end
    }
end
```

`on:` 定义自定义的上下文。必须把上下文的名称传给 valid?、invalid? 或 save 才能触发自定义的上下文。

```ruby
class Person < ApplicationRecord
  validates :email, uniqueness: true, on: :account_setup
  validates :age, numericality: true, on: :account_setup
end

person = Person.new
person.valid?(:account_setup) # 会执行上述两个验证，但不保存记录。
person.save(context: :account_setup) # 在保存之前在 account_setup 上下文中验证 person。显式触发时，可以只使用某个上下文验证，也可以不使用某个上下文验证。
```

### 严格验证

数据验证还可以使用严格模式，当对象无效时抛出 `ActiveModel::StrictValidationFailed` 异常，还可以通过 :strict 选项指定抛出什么异常：

```ruby
class Person < ApplicationRecord
  validates :name, presence: { strict: true }
end

Person.new.valid?  # => ActiveModel::StrictValidationFailed: Name can't be blank
```

```ruby
class Person < ApplicationRecord
  validates :token, presence: true, uniqueness: true, strict: TokenGenerationException
end

Person.new.valid?  # => TokenGenerationException: Token can't be blank
```

### 条件验证

* `:if, :unless` 的值可以是符号、字符串、Proc 或数组。
* 同一个条件会用在多个验证上，这时可以使用 `with_options` 方法

```ruby
class Order < ApplicationRecord
  validates :card_number, presence: true, if: :paid_with_card?

  def paid_with_card?
    payment_type == "card"
  end
end

class Account < ApplicationRecord
  validates :password, confirmation: true, unless: Proc.new { |a| a.password.blank? }
end

class User < ApplicationRecord
  with_options if: :is_admin? do |admin|
    admin.validates :password, length: { minimum: 10 }
    admin.validates :email, presence: true
  end
end
```

### 自定义验证

1. 自定义的验证类继承自 `ActiveModel::Validator`，必须实现 `validate` 方法，其参数是要验证的记录，然后验证这个记录是否有效。自定义的验证类通过 `validates_with` 方法调用。
2. 自定义的验证类中验证单个属性，最简单的方法是继承 `ActiveModel::EachValidator` 类。此时，自定义的验证类必须实现 `validate_each` 方法。
3.

```ruby
class MyValidator < ActiveModel::Validator
  def validate(record)
    unless record.name.starts_with? 'X'
      record.errors[:name] << 'Need a name starting with X please!'
    end
  end
end

class Person
  include ActiveModel::Validations
  validates_with MyValidator
end
```

```ruby
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors[attribute] << (options[:message] || "is not an email")
    end
  end
end

class Person < ApplicationRecord
  validates :email, presence: true, email: true
end
```

### 处理验证错误

* `errors[]` 用于获取某个属性上的错误消息，返回结果是一个由该属性所有错误消息字符串组成的数组，每个字符串表示一个错误消息。如果字段上没有错误，则返回空数组。
* `add` 方法用于手动添加某属性的错误消息，它的参数是属性和错误消息
* `errors.full_messages`（或等价的 errors.to_a）方法以对用户友好的格式显示错误消息
* `errors.add` 方法可以为返回的错误详情散列指定验证程序类型，如果想提升错误详情的信息量，可以为 `errors.add` 方法提供额外的键，指定不允许的字符。
* `errors[:base]` 错误消息可以添加到整个对象上，而不是针对某个属性
* 在无效的对象上调用 `errors.clear` 方法后，对象还是无效的，虽然 errors 集合为空了
* `size` 方法返回对象上错误消息的总数

```ruby
class Person < ApplicationRecord
  def a_method_used_for_validation_purposes
    errors.add(:name, "cannot contain the characters !@#%*()_-+=")
  end

  def a_method_used_for_validation_purposes
    errors.add(:name, :invalid_characters, not_allowed: "!@#%*()_-+=")
  end
end

person.valid?
person.errors.messages #=> {:name=>["can't be blank", "is too short (minimum is 3 characters)"]}
person.errors[:name] #=> ["is too short (minimum is 3 characters)"]
person.errors.full_messages #=> ["Name cannot contain the characters !@#%*()_-+="]
person.errors.details[:name] #=> [{error: :invalid_characters}]
```

## 相关链接

* [Rails Guides](https://yuque.com/ruby-china/rails-guides)
* [Ruby on Rails 指南](https://ruby-china.github.io/rails-guides/)
