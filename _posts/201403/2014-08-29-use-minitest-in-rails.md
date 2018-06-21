---
layout: post
title:  "Rails 搭配 minitest 写测试"
date:   2014-08-29 11:20:00
categories: rails
tags: testing
author: "Victor"
---

更新于：2017年5月13日

## 概览

`Test::Unit` 是 Rails 默认测试框架的名称，自 Rails 4 开始，其核心功能和断言被 `Minitest` 取代。当你 `require 'test/unit'` 的时候，其实底层真正使用的是 `MiniTest::Unit`。`MiniTest::Spec` 也提供了类似 RSpec 的语法。

### MiniTest::Unit::TestCase

```ruby
require 'minitest/autorun'

class TestHipster < MiniTest::Unit::TestCase
  def setup
    @hipster = Hipster.new
    @labels  = Array.new
    @traits  = ["silly hats", "skinny jeans"]
  end

  def teardown
    @hipster.destroy!
  end

  def test_for_helvetica_font
    assert_equal "helvetica!", @hipster.preferred_font
  end

  def test_not_mainstream
    refute @hipster.mainstream?
  end
end
```

所有的 `assert` 都有对应的 `refute` 断言。

| Assertion	| Example |
| ------------- |-------------|
| `assert	assert` | `@traits.any?, "empty subjects"` |
| `assert_empty` | `assert_empty @labels` |
| `assert_equal` | `assert_equal 2, @traits.size` |
| `assert_in_delta` | `assert_in_delta @traits.size, 1,1` |
| `assert_in_epsilon` | `assert_in_epsilon @traits.size, 1, 1` |
| `assert_includes` | `assert_includes @traits, "skinny jeans"` |
| `assert_instance_of` | `assert_instance_of Hipster, @hipster` |
| `assert_kind_of` | `assert_kind_of Enumerable, @labels` |
| `assert_match` | `assert_match @traits.first, /silly/` |
| `assert_nil` | `assert_nil @labels.first` |
| `assert_operator` | `assert_operator @labels.size, :== , 0` |
| `assert_output` | `assert_output("Size: 2") { print "Size: #{@traits.size}"}` |
| `assert_raises` | `assert_raises(NoMethodError) { @traits.foo }` |
| `assert_respond_to` | `assert_respond_to @traits, :count` |
| `assert_same` | `assert_same @traits, @traits, "It's the same object silly"` |
| `assert_send` | `assert_send [@traits, :values_at, 0]` |
| `assert_silent` | `assert_silent { "no stdout or stderr" }` |
| `assert_throws` | `assert_throws(Exception,'is empty') {throw Exception if @traits.any?}` |

### MiniTest#stub

`Minitest` 提供一个简单的 `stub` 方法，我们可以模拟外部依赖返回一些预设的值。


```ruby
require 'minitest/autorun'

describe Hipster, "Demonstrates stubbing with Minitest" do

  let(:hipster) { Hipster.new }

  it "trendy if time is now" do
    assert hipster.trendy? DateTime.now
  end

  it "it is NOT trendy if 2 weeks has past" do
    DateTime.stub :now, (Date.today.to_date - 14) do
      refute hipster.trendy? DateTime.now
    end
  end
end
```

### MiniTest::Mock

```ruby
require 'minitest/autorun'

# Make all of our Twitter updates hip
class Twipster
  def initialize(twitter)
    @twitter = twitter # A Twitter API client
  end

  def tweet(message)
    @twitter.update("#{message} #lolhipster")
  end
end

# Uses Mock#expect and Mock#verify
describe Twipster, "Make every tweet a hipster tweet." do
  before do
    @twitter  = MiniTest::Mock.new # Mock our Twitter API client
  end

  let(:twipster) { Twipster.new(@twitter) }
  let(:message) { "Skyrim? Too mainstream."}

  it "should append a #lolhipster hashtag and update Twitter with our status" do
    @twitter.expect :update, true, ["#{message} #lolhipster"]
    @twipster.tweet(message)

    assert @twitter.verify # verifies tweet and hashtag was passed to `@twitter.update`
  end
end
```

## Rails

### 目录结构

`Minitest` 不关心你如何组织测试文件的目录结构和名称，但是 Rails 开发团队有一些相关的建议。

```
test/
  test_helper.rb
  controllers/
  fixtures/
  helpers/
  integration/   <- Capybara tests go here
  mailers/
  models/
  unit/lib/      <- Tests for your /lib code
```

### 相关 Gem 推荐

```ruby
group :development do
  gem "guard", ">= 2.2.2", :require => false
  gem "guard-minitest", :require => false
  gem "rb-fsevent", :require => false
  gem "terminal-notifier-guard", :require => false
end

group :test do
  gem "capybara"
  gem "connection_pool"
  gem "launchy"
  gem "minitest-reporters"
  gem "mocha"
  gem "poltergeist"
  gem "shoulda-context"
  gem "shoulda-matchers", ">= 3.0.1"
  gem "test_after_commit"
end
```

```ruby
guard :minitest, :spring => true do
  watch(%r{^app/(.+)\.rb$})                               { |m| "test/#{m[1]}_test.rb" }
  watch(%r{^app/controllers/application_controller\.rb$}) { "test/controllers" }
  watch(%r{^app/controllers/(.+)_controller\.rb$})        { |m| "test/integration/#{m[1]}_test.rb" }
  watch(%r{^app/views/(.+)_mailer/.+})                    { |m| "test/mailers/#{m[1]}_mailer_test.rb" }
  watch(%r{^app/workers/(.+)\.rb$})                       { |m| "test/unit/workers/#{m[1]}_test.rb" }
  watch(%r{^lib/(.+)\.rb$})                               { |m| "test/unit/lib/#{m[1]}_test.rb" }
  watch(%r{^lib/tasks/(.+)\.rake$})                       { |m| "test/unit/lib/tasks/#{m[1]}_test.rb" }
  watch(%r{^test/.+_test\.rb$})
  watch(%r{^test/test_helper\.rb$}) { "test" }
end
```

## 相关文章

* [Minitest Quick Reference](http://www.mattsears.com/articles/2011/12/10/minitest-quick-reference/)
* [TDD with MiniTest and EventManager](http://tutorials.jumpstartlab.com/academy/workshops/testing_event_manager.html)
* [minimalicous-testing-in-ruby-1-9](http://blog.arvidandersson.se/2012/03/28/minimalicous-testing-in-ruby-1-9)
