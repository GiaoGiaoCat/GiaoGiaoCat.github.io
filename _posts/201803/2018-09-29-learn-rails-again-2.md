---
layout: post
title:  "重读 Rails Guide 2"
date:   2018-09-29 13:00:00

categories: rails
tags: learningnote
author: "Victor"
---

## Active Record 回调

回调是在对象生命周期的某些时刻被调用的方法。通过回调，我们可以编写在创建、保存、更新、删除、验证或从数据库中加载 Active Record 对象时执行的代码。

```ruby
class User < ApplicationRecord
  before_create do
    self.name = login.capitalize if name.blank?
  end

  after_validation :set_location, on: [ :create, :update ]

  private

    def set_location
      self.location = LocationService.query(self)
    end
end
```

**如果模型中的关联有 `dependent: :destroy` 配置项，那么 `before_destroy` 需要声明在其之前，或者在 `before_destroy` 宏声明上添加 `prepend: true` 选项。**

```ruby
class User < ActiveRecord::Base
  has_many :authorizations, dependent: :delete_all
  before_destroy :remove_external_account

  def remove_external_account
    authorization_uuid = authorizations.last.uuid
    return unless authorization_uuid
    Rails.log.info(‘ [DebugCode] before “call Sidekiq worker”’)
    RemoveExternalAccountWorker.perform_async(user.id, authorization_uuid)
  end
end
```

```bash
ser Load (0.4ms) SELECT “users”.* FROM “users” WHERE “users”.”id” = 1 LIMIT 1 [[“id”, 1]]
SQL (0.6ms) DELETE FROM “authorizations” WHERE “authorizations”.”user_id” = 1 [[“user_id”, 1]]
SQL (0.4ms) DELETE FROM “users” WHERE “users”.”id” = 1 [[“id”, 1]]
[DebugCode] before “call Sidekiq worker”
Authorization Load (0.4ms) SELECT “authorizations”.”uuid” FROM “authorizations” WHERE “authorizations”.”user_id” = 1 AND “authorizations”.”provider” = 0 ORDER BY “authorizations”.”id” ASC LIMIT 1 [[“user_id”, 1], [“provider”, 0]]
```

这里 authorizations 先被删除了，然后才执行到 `before_destroy :remove_external_account` 这里，而 `remove_external_account` 想读取 authorizations 的数据，就会出错。正确写法如下：

```ruby
class User < ActiveRecord::Base
  has_many :authorizations, dependent: :delete_all
  before_destroy :remove_external_account, prepend: true
end
```

### after_initialize 和 after_find 回调

* 不管是通过直接使用 `new` 方法还是从数据库加载记录，都会调用 `after_initialize` 回调
* 从数据库中加载记录时，会调用 `after_find` 回调
* 会先调用 `after_find`，再调用 `after_initialize`
* 不存在 `before_find` 和 `before_initialize`

### after_touch

在对象执行 `touch` 方法之后会触发该回调，可以配合 `belongs_to` 的 `touch: true`。

### 跳过回调

* decrement
* decrement_counter
* delete
* delete_all
* increment
* increment_counter
* toggle
* update_column
* update_columns
* update_all
* update_counters

Rails 手册上给的建议是慎用这些方法，因为一些重要的业务逻辑可能保存在回调中。

模型中的验证，回调和数据库操作都被包裹在事务中，排队执行，期间任何回调引发异常都会造成整个回调链的回滚，当然也可以手动使用 `throw :abort` 来停止回调链。

需要注意的地方是当回调链停止后，Rails 会重新抛出除 `ActiveRecord::Rollback` 和 `ActiveRecord::RecordInvalid` 之外的其他异常。这可能导致那些预期返回 true 或 false 的方法，比如 `save` 和 `update_attributes` 代码出错。

### 条件回调

```ruby
class Order < ApplicationRecord
  before_save :normalize_card_number, if: Proc.new { paid_with_card? }
end
```

而根据 DRY 原则，如果我们创建的回调可以被其它模型使用，最好是单独写回调类。

```ruby
class PictureFileCallbacks
  def self.after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end

class PictureFile < ApplicationRecord
  after_destroy PictureFileCallbacks
end
```

### 事务回调

`after_commit` 和 `after_rollback` 这两个回调会在数据库事务完成时触发。和 `after_save` 回调区别在于它们在数据库变更已经提交或回滚后才会执行，常用于 Active Record 模型需要和数据库事务之外的系统交互的场景。

如果 `after_destroy` 回调执行后应用引发异常，事务就会回滚，文件会被删除，模型会保持不一致的状态。通过使用 after_commit 回调，我们可以解决这个问题：

```ruby
class PictureFile < ApplicationRecord
  after_commit :delete_picture_file_from_disk, on: :destroy

  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

由于只在执行创建、更新或删除动作时触发 after_commit 回调是很常见的，这些操作都拥有别名：

* after_create_commit
* after_update_commit
* after_destroy_commit

### 注意

1. 在事务中创建、更新或删除模型时会调用 `after_commit` 和 `after_rollback` 回调。然而，如果其中有一个回调引发异常，异常会向上冒泡，后续 `after_commit` 和 `after_rollback` 回调不再执行。因此，如果回调代码可能引发异常，就需要在回调中救援并进行适当处理，以便让其他回调继续运行。
2. `after_commit` 和 `after_rollback` 内部执行的代码，本身不包含在事务中。
3. 在同一个模型中定义 `after_create_commit` 和 `after_update_commit`，只有最后定义的会生效

```ruby
# Wrong
class User < ApplicationRecord
  after_create_commit :log_user_saved_to_db
  after_update_commit :log_user_saved_to_db

  private
  def log_user_saved_to_db
    puts 'User was saved to database'
  end
end

# Right
class User < ApplicationRecord
  after_commit :log_user_saved_to_db, on: [:create, :update]
end
```

## Active Record 关联

## Active Record 查询接口

## 相关链接

* [Rails Guides](https://edgeguides.rubyonrails.org/index.html)
* [Ruby on Rails 指南](https://ruby-china.github.io/rails-guides/)
* [Caution when using before_destroy with model association](https://medium.com/appaloosa-store-engineering/caution-when-using-before-destroy-with-model-association-71600b8bfed2)
* [Callbacks](https://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)
