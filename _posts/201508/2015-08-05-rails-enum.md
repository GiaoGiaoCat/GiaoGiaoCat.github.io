---
layout: post
title:  "How to use i18n with Rails 4 enums"
date:   2015-08-05 13:00:00
categories: rails
tags: tip
author: "Victor"
---

利用 Enums，再也不需要用多个布尔值来表示对象的状态了。

```ruby
class Bug < ActiveRecord::Base
  # Relevant schema change looks like this:
  #
  # create_table :bugs do |t|
  #   t.column :status, :integer, default: 0 # defaults to the first value (i.e. :new)
  # end

  enum status: [ :new, :assigned, :in_progress, :resolved, :rejected, :reopened ]

  belongs_to :assignee, class_name: 'Developer'

  def assignee=(developer)
    if developer && self.new?
      self.status = :assigned
    else
      self.status = :new
    end

    super
  end
end

Bug.resolved            # => a scope to find all resolved bugs
bug.resolved?           # => check if bug has the status :resolved
bug.resolved!           # => update! the bug with status set to :resolved

bug.status              # => a symbol describing the bug's status

# bug.update! status: 3
bug.status = :resolved  # => set the bug's status to :resolved
bug.status = 'resolved' # => set the bug's status to :resolved

# bug.update! status: nil
bug.status = nil
bug.status.nil? # => true
bug.status      # => nil
```

在内部，与这些状态在数据库中相对应的是整数值，以节省空间。同样值得一提的是，`enum` 宏所添加的方法是通过 `mix-in` 一个 module 来实现的。这意味着你可以方便地在 model 中重写它们并使用 `super` 来调用原来的实现。

## 使用该特性时还有如下一些注意事项：

### 所以一旦定义完 `enum` 的 `symbol` 后，你就不应该再去改动其顺序了。

不要被其名字所迷惑，在一些数据库中并不使用 `ENUM` 类型来实现该特性。状态和其对应的整数值的匹配是通过 model 文件来维护的。所以一旦定义完 `enum` 的 `symbol` 后，你就不应该再去改动其顺序了。可以明确地指定 mapping 来删除不再使用的状态：

```ruby
class Bug < ActiveRecord::Base
  enum status: {
    new: 0,
    in_progress: 2,
    resolved: 3,
    rejected: 4,
    reopened: 5
  }
end
```

### 避免在一个类的不同 enum 中使用相同的名字！这样会使 Active Record 很迷茫！

```ruby
class Bug < ActiveRecord::Base
  enum status: [ :new, ... ]
  enum code_review_status: [ :new, ... ] # WARNING: Don't do this!
end
```

### 用 scope 来查询 enum 字段：

```ruby
Conversation.active
Conversation.where(status: [:active, :archived])
Conversation.where.not(status: :active)
```

## 如何配合 SimpleForm 和 I18N

```ruby
# model
class User < ActiveRecord::Base
  enum role: [:admin, :member]
end
```

```ruby
# view
= field.input :gender, collection: t_options_for_select(User::Profile, :gender), include_blank: false, wrapper: :horizontal_form
```

```ruby
# helper
def t_options_for_select(class_obj, enums_name)
  enums = class_obj.send(enums_name.to_s.pluralize)
  enums.keys.map {|enum| t_enum_option_for_select(enum, class_obj.name.underscore.to_sym, enums_name) }
end

def t_enum_option_for_select(enum, class_name, enum_name)
  [t(enum, scope: [:enumerize, class_name, enum_name]), enum]
end
```

```yaml
# config/locales/enumerize.yml
zh:
  enumerize:
    user/profile:
      gender:
        male: "男"
        female: "女"
```

## 相关阅读

* [Rails 4.1 的新特性（译）](https://ruby-china.org/topics/18504)
* [ActiveRecord::Enum](http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html)
* [Select enum from form to set role](http://stackoverflow.com/questions/24193814/select-enum-from-form-to-set-role)
