---
layout: post
title:  "A Tip of Ruby Initialize Method"
date:   2014-09-12 17:10:00
categories: ruby
tags: tip
author: "Victor"
---

在遇到很多变量在 initialize 方法中一次性设置的时候，可以用下面的方法

```ruby
class Template
  STRING_ATTRIBUTES = %i(title text logo logo_url).freeze
  BOOLEAN_ATTRIBUTES = %i(is_ring is_vibrate is_clearable).freeze

  attr_accessor *STRING_ATTRIBUTES, *BOOLEAN_ATTRIBUTES

  def initialize
    super

    STRING_ATTRIBUTES.each { |attr| instance_variable_set("@#{attr}", '') }
    BOOLEAN_ATTRIBUTES.each { |attr| instance_variable_set("@#{attr}", true) }
  end
end
```
