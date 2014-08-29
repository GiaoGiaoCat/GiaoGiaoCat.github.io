---
layout: post
title:  "Rails 搭配 minitest 写测试"
date:   2014-08-29 11:20:00
categories: rails
tags: testing
author: "Victor"
---

在 Ruby 圈里面 Test::Unit 和 RSpec 是最流行的测试库。Test::Unit 是 Ruby 的标准库，但自从 Ruby 1.9 and 2.0，它被 MiniTest 替代。

这种替换对于使用 Test::Unit 的人来说是透明的，因为 MiniTest 会覆盖且兼容它。当你 require 'test/unit' 的时候，其实底层真正使用的是
 MiniTest::Unit 。

对于喜欢 RSpec 语法的朋友，MiniTest::Spec 也提供了类似的语法。

## Setup

Gemfile

```ruby
group :test do
  gem 'minitest-rails'
  gem 'minitest-rails-capybara'
  # gem 'minitest-colorize' # Do NOT use this gem! because of the 'Minitest vs MiniTest for Unit Testing (completely broken)'
  gem 'minitest-focus'
end
```

如果 ```bundle install``` 无法安装相关 gem 的话，可以试试如下命令

```ruby
# http://stackoverflow.com/questions/4118055/rails-bundler-doesnt-install-gems-inside-a-group
bundle install --without nothing #(just clearing cache, now all the gems to be loaded into the ruby loadpath)
```

```ruby
rails g minitest:install
```

修改 test/test_helpe.rb 增加 MiniTest 用到的 lib 和一些辅助方法

```ruby
ENV["RAILS_ENV"] = "test"
require File.expand_path("../../config/environment", __FILE__)
require "rails/test_help"
require "minitest/rails"

# To add Capybara feature tests add `gem "minitest-rails-capybara"`
# to the test group in the Gemfile and uncomment the following:
# require "minitest/rails/capybara"

# Uncomment for awesome colorful output
require "minitest/pride"

require 'minitest/focus'

class ActiveSupport::TestCase
    ActiveRecord::Migration.check_pending!

    # Setup all fixtures in test/fixtures/*.(yml|csv) for all tests in alphabetical order.
  #
  # Note: You'll currently still have to declare fixtures explicitly in integration tests
  # -- they do not yet inherit this setting
  fixtures :all

  # Add more helper methods to be used by all tests here...
  def self.prepare
    # Add code that needs to be executed before test suite start
  end
  prepare

  def setup
    # Add code that need to be executed before each test
  end

  def teardown
    # Add code that need to be executed after each test
  end
end
```

修改 config/application.rb

```ruby
# require 'rails/all'
# Pick the frameworks you want:
require 'rake/testtask'
require "minitest/rails/railtie"
```

```ruby
class Application < Rails::Application
    # Config minitest-rails
    # https://github.com/blowmage/minitest-rails#usage
    config.generators do |g|
      g.test_framework :minitest, spec: false, fixture: true
    end
end
```

## Create Test db

Rails 4.1.X 号称已经删除 rake db:test:prepare ，而 rake test 会自动 load 数据结构，目前看来是不好使的。

```
rake db:schema:dump
rake db:test:prepare
rake test
```

## Run Test

```ruby
rake minitest # Run default tests
rake minitest:all # Run all tests
rake minitest:all:quick # Run all tests, ungrouped for quicker execution
rake minitest:controllers # Runs tests under test/controllers
rake minitest:helpers # Runs tests under test/helpers
rake minitest:models # Runs tests under test/models
rake test # Run default tests
```

## 相关文章

* [Testing Rails with MiniTest](http://blog.crowdint.com/2013/06/14/testing-rails-with-minitest.html)
* [TDD with MiniTest and EventManager](http://tutorials.jumpstartlab.com/academy/workshops/testing_event_manager.html)
* [A MiniTest::Spec Tutorial: Elegant Spec-Style Testing That Comes With Ruby](http://www.rubyinside.com/a-minitestspec-tutorial-elegant-spec-style-testing-that-comes-with-ruby-5354.html)
* [minimalicous-testing-in-ruby-1-9](http://blog.arvidandersson.se/2012/03/28/minimalicous-testing-in-ruby-1-9)
