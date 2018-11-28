---
layout: post
title:  "Rails 项目避免用户使用弱密码"
date:   2018-11-27 12:00:00

categories: rails
tags: tip
author: "Victor"
---

并不是所有用户都和你一样喜欢使用 1Password 或 KeePass 这样的工具来管理自己的密码。普通用户都是喜欢使用自己易于记录的密码，或者所有网站都用同样的密码。

有一种观点：**教育用户并保护他们的安全是开发者的责任，即使这对用户来说可能有点不愉快。**

基于这一观点，开发者会禁止用户使用易于被暴力猜解的密码，比如 `12345678` 或者 `qwerty` 这种。

密码策略是一组规则，旨在通过鼓励用户使用强密码并正确使用它们来增强计算机安全性。

对于 Rails 应用来说，只需要在 User 模型中添加一个自定义的验证方法，就可以为应用添加简单的密码策略。

```ruby
validate :password_complexity
def password_complexity
  return if password.blank? || password =~ /^(?=.*?[A-Z])(?=.*?[a-z])(?=.*?[0-9])(?=.*?[#?!@$%^&*-]).{8,70}$/
  errors.add :password, "Complexity requirement not met. Length
    should be 8-70 characters and include: 1 uppercase, 1 lowercase,
    1 digit and 1 special character"
end
```

除此之外也可以选择 [strong_password](https://github.com/bdmac/strong_password) 这样的 gem 来实现此功能。

## 相关链接

* [5 security issues in Ruby on Rails apps from real life](https://frontdeveloper.pl/2018/10/5-security-issues-in-ruby-on-rails/)
