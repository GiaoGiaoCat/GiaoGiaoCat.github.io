---
layout: post
title:  "Skipping Validations in Ruby On Rails"
date:   2016-08-10 10:30:00
categories: rails
tags: tip
author: "Victor"
---

当模型划分的足够好的时候，下面的技术应该用不上。

## Skip All Validations

```ruby
def create
  @product = Product.new(product_params)
   if @product.save(validate: false)
    redirect_to products_path, notice: "#{@product.name} has been created."
  else
    render 'new'
  end
end

def update
  @product = Product.find(params[:id])
  @product.attributes = product_params

  if @product.save(validate: false)
    redirect_to products_path, notice: "The product \"#{@product.name}\" has been updated. "
  else
    render 'edit'
  end
end
```

## Skipping Individual Validations

```ruby
class Product < ActiveRecord::Base
  validates :name, presence: true, uniqueness: true, unless: :skip_name_validation
  validates :price, presence: true, numericality: { greater_than: 0 }, unless: :skip_price_validation

  attr_accessor :skip_name_validation, :skip_price_validation
end
```

```ruby
def create
   @product = Product.new(product_params)
   @product.skip_name_validation = true
   if @product.save
    redirect_to products_path, notice: "#{@product.name} has been created."
  else
    render 'new'
  end
end

def update
  @product = Product.find(params[:id])
  @product.attributes = product_params

  @product.skip_price_validation = true

  if @product.save
    redirect_to products_path, notice: "The product \"#{@product.name}\" has been updated. "
  else
    render 'edit'
  end
end
```

## 原文

* [Skipping Validations in Ruby On Rails](https://richonrails.com/articles/skipping-validations-in-ruby-on-rails)
