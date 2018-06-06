---
layout: post
title:  "Rails 5 Controller Tests"
date:   2017-05-15 11:30:00
categories: rails
tags: testing
author: "Victor"
---

## controller tests 不再是 unit tests

Rails 5 的 `controller tests` 现在成了 `integration tests`。这意味着好多内部交互的功能都被移除了。如果你过去的项目因为某些原因撸了一大堆 `controller tests`，那你会发现有一大堆过去在测试代码里面用的 hacks 技巧都不能再用了。

### `ActionController::TestCase` is deprecated

Rails 5 的 `controller tests` 继承自 `ActionDispatch::IntegrationTest`。

```ruby
class ProductsControllerTest < ActionController::TestCase
  def test_index_response
    #...
  end
end

# Rails 5
class ProductsControllerTest < ActionDispatch::IntegrationTest
  def test_index
    #...
  end
end
```

### URLS, not actions, are required.

Rails 5 要求我们在测试中使用 URL Helper 来指定路由。

```ruby
# Rails 4
get :index
get :index, { id: 1 }

# Rails 5
get root_path
get posts_path(@post)
```

### Use of Keywords arguments in HTTP request methods

我们需要明确的给出 `params` 参数关键字。

Rails 4.x, 我们可以一次传递各种参数，比如： `params`, `flash messages` 和 `session variables`。但是 Rails 5 要求我们必须明确给定参数名称。

```ruby
# Rails 4
get :show, { id: user.id }, { notice: 'Welcome' }, { admin: user.admin? }

# Rails 5
post posts_path, params: { post: { title: "First post", body: "This is the body"} }
```

### No more access to `request`

`request` 对象被移除。

```ruby
# Rails 4
request.env["HTTP_AUTHORIZATION"] = something....

# Rails 5
get admin_path, headers: {'HTTP_AUTHORIZATION' => something }
```

### No more access to `session`

你不能再利用给 `session` 赋值的方式实现授权之类的操作了。现在你必须真实的触发登录页面，[DHH 对此的解释](https://github.com/rails/rails/issues/23386)。

```ruby
def sign_in_as(name)
  post login_url, params: { sig: users(name).perishable_signature )
end

setup { sign_in_as 'david' }
```

这看起来就像我们过去在 `integration tests` 中干的事一样。没错，因为现在 `controller tests` 就是 `integration tests` 了。没啥可惊喜的，但是为了兼容，你不得不修改你的一大堆测试代码。

### No more access to `assigns` and `assert_template`

不能再使用 `assigns` 来访问控制器中的实例变量，也不能使用 `assert_template` 来验证该 `action` 渲染的是指定的某个 `template`。它们都被移除了。你可以换用 `assert_select` 来判断页面中或响应中的节点内容，以验证结果。

原因是这样的，老大们觉得实例变量和渲染哪个模板那是控制器内部实现的细节，新版的 `controller tests` 不应该关注这些东西。按照 Rails 核心团队的说法，`controller tests` 应该关注的是 `action` 的结果（返回的 `cookies` 和 HTTP 状态码正确）。

## 结论

Rails 中再也没有功能测试 `controller tests` 这一说法了，现在它成了集成测试 `integration tests`。那以前的 `integration tests` 怎么办呢？将来再说。

## 相关阅读

* [Upgrading Rails 5 Controller Tests](http://blog.napcs.com/2016/07/03/upgrading-rails-5-controller-tests/)
* [Changes to test controllers in Rails 5](http://blog.bigbinary.com/2016/04/19/changes-to-test-controllers-in-rails-5.html)
* [Upgrading Rails 4 Controller Tests to Rails 5](http://engineering.appfolio.com/appfolio-engineering/2018/5/25/upgrading-rails-4-controller-tests-to-rails-5)
