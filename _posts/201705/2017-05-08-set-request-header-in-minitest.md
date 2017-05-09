---
layout: post
title:  "Best way to set a header to all tests using MiniTest"
date:   2017-05-08 11:30:00
categories: rails
tags: testing
author: "Victor"
---

在使用 Rails 构件 API 应用的时候，通常会把用户的 token 放在 `request.header` 中。在测试时候，我们可以通过下面的方式，给所有的 `controller test` 文件都加上 token

```ruby
# test/lib/custom_header_setup.rb
module CustomHeaderSetup                                                       
  def before_setup
    super
    @request.headers['Custom-Header'] = 'Custom Value'
  end
  ActionController::TestCase.send(:include, self)
end
```

`test/test_helper.rb` 引入该文件：

```ruby
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'
require 'support/custom_header_setup'

class ActiveSupport::TestCase
  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end
```
