---
layout: post
title:  "Rails 的乐观锁 Optimistic Locking"
date:   2016-08-09 10:30:00
categories: rails
tags: tip
author: "Victor"
---

也许会遇到两个人在接近的时间，同时去后台编辑一条记录的情况。可以通过 Rails 的乐观锁机制或者使用 updated_at 字段来判断用户要更新的对象，在数据库中的版本是否有变化。

## Rails 内置的 `lock_version` 字段实现乐观锁


```ruby
# /db/migrations/20120820000000_add_lock_version_to_products.rb
class AddLockVersionToProducts < ActiveRecord::Migration
  def change
    add_column :products, :lock_version, :integer, default: 0, null: false
  end
end

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

```erb
<%= f.hidden_field :lock_version %>
```

## 使用 `updated_at` 来实现乐观锁

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

## 相关阅读

* [What is Optimistic Locking](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html)

