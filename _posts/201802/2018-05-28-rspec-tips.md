---
layout: post
title:  "RSpec tips"
date:   2018-05-28 12:00:00

categories: rails
tags: testing
author: "Victor"
---

新公司的项目用 RSpec，从 MiniTest 转过来的我有点懵，先撸一些 Tips 方便自己用吧。

## Model Specs
### Test Validation Errors with RSpec

在进行单元测试中，有时候需要验证一些自定义的 model validations

```ruby
# app/models/developer.rb
validates :email, format: { with: /foo/ }

# spec/models/developer_spec.rb
developer.email = 'bar'
expect(developer).to_not be_valid
expect(developer.errors.message[:email]).to eq ['is invalid']
```

## 相关阅读

* [TODAY I LEARNED](https://til.hashrocket.com/testing)
* [Relish RSpec](https://relishapp.com/rspec/)
