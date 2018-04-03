---
layout: post
title:  "Better Gemfile"
date:   2018-03-23 15:05:00

categories: rails
tags: tip
author: "Victor"
---

Gemfile.lock 让 developer machine, CI server, production environment 保持相同的依赖环境。

首先要知道，我们应该尽可能减少对 gem 的依赖。因为：

Some software engineering rules:

* No code runs faster than no code.
* No code has fewer bugs than no code.
* No code uses less memory than no code.
* No code is easier to understand than no code.

Kill those dependencies. Your gems and apps will be better for it.

### BAD

devise and figaro are locked without telling me why.

```ruby
gem "rails", "~> 4.0.10"

gem "pg"
gem "delayed_job_active_record"
gem "devise", "~> 3.1.2"
gem "figaro", github: "laserlemon/figaro"
gem "will_paginate"
gem "bootstrap-will_paginate"
gem "simple_form"
gem "honeybadger"
gem "draper"
gem "stripe"
```

### BETTER

```ruby
gem "devise", "~> 3.1.2"  # 3.2 removed token auth
gem "figaro", github: "laserlemon/figaro"  # until 1.0.0 is released
gem "heroku", github: "heroku/heroku" # until 3.10.6 is released.
                                      # See: https://github.com/heroku/heroku/issues/1201
```

```ruby
source "https://rubygems.org"

ruby "2.1.2"

gem "rails", "~> 4.1.6"

gem "pg"

# Asset Pipeline
gem "coffee-rails"
gem "jquery-rails"
gem "jquery-ui-rails"
gem "mapbox-rails"
gem "sass-rails"
gem "uglifier"
gem "wysihtml5-rails"

gem "active_model_serializers"
gem "active_record_union"
gem "acts_as_geocodable"
gem "audited-activerecord"
gem "awesome_nested_set"
gem "color"
gem "countries"
gem "csv_builder"
gem "dalli"
gem "delayed_job"

group :development, :test do
  gem "guard-rspec"
  gem "launchy"
  gem "pry-rails"
  gem "rspec-rails"
end
```

## 结论

* 引入之前，先想一下是否真的需要这个 gem
  * 谁引入谁负责
* 保持文件简单容易理解，不要使用 Ruby 技巧
  * 保持 Ruby 风格统一，例如：quoted delimiter, hash syntax
* Specifying a Ruby Version
* **Don’t specify versions until you need to**
* **Add comments telling you why**
  * 没有被注释掉的 gem
  * 注释在 gem 的后面，而不要在上面
* **Alphabetize your Gemfile, occasionally breaking up groups like asset pipeline gems**
  * 使用 group 来分组，并使用 block 的方式，避免使用 single line group `gem 'web-console', group: :development`
  * group 的排序应该是 `default(general), production, assets, test, development`
* **Update often**
* gem versions
  * Use a *pessimistic version* in the Gemfile for gems that follow semantic versioning, such as rspec, factory_girl, and capybara.
  * Use a *versionless* Gemfile declarations for gems that are safe to update often, such as pg, thin, and debugger.
  * Use an *exact version* in the Gemfile for fragile gems, such as Rails.

```ruby
# exact version
gem 'nokogiri', '1.0.3'
gem 'webrat', '0.3.1'

# pessimistic version
gem 'nokogiri', '~> 1.0.3'
gem 'webrat', '~> 0.3.1'

# versionless
gem 'nokogiri'
gem 'webrat'
```

遗留项目升级的实践方案：

1. 每周一执行 `bundle outdated` 捡出 1-3 个可以升级的 gem
2. 阅读 changelog
3. 按照下文 *You Should Update One Gem at a Time with Bundler* 的方式进行升级更新
4. 维护一份含有 gems 功能和链接的说明文档

### You Should Update One Gem at a Time with Bundler

1. Cut a new branch: `git checkout -b upgrade-gems`
2. 使用 `bundle update gemname` 仍然会带来一些同是升级依赖的问题，但这是一种很好的进步
3. git diff Gemfile.lock
4. 如果升级造成了严重问题，就锁定到最后一个可正常工作的版本并注明原因
5. Once the tests pass, commit and push the branch, which runs the tests again on our continuous integration server and then deploy it to a staging environment.

* Do this often and you’ll always have up-to-date gems.
* when you’re only updating a few gems at a time, it is much easier to find problems with upgrades.

## 其它

### 一些有用的命令

```bash
gem dependency <gemname> # 查看 gem 的依赖
bundle show <gemname> # 查看 gem 的安装路径
bundle outdated # 检查哪些 gem 需要更新
```

### 关于 Semantic Versioning

* `~> 5.0.0` is equivalent to `>= 5.0.0 <5.1`
* `~> 2.3` will allow `2.3.1, 2.5.0` but not `3.0.0`

### Gems 推荐

* [bullet](https://github.com/flyerhzm/bullet) help to kill N+1 queries and unused eager loading
* [peek](https://github.com/peek/peek) Take a peek into your Rails applications
* [brakeman](https://github.com/presidentbeef/brakeman) Ruby 静态代码安全检测工具
* [bundle-audit](https://github.com/rubysec/bundler-audit) 检查你用的 gem 是否有安全补丁需要升级
* [rails_best_practices](https://github.com/flyerhzm/rails_best_practices) 检测 Rails 项目的代码质量
* [traceroute](https://github.com/amatsuda/traceroute) 无效路由和控制器方法检测
* [bundler-stats](https://github.com/jmmastey/bundler-stats) 帮助检查 gem 的依赖

## 相关链接

* [Semantic Versioning 2.0.0](https://semver.org/)
* [You Should Update One Gem at a Time with Bundler. Here’s How.](https://ilikestuffblog.com/2012/07/01/you-should-update-one-gem-at-a-time-with-bundler-heres-how/)
* [How we write a Gemfile](https://collectiveidea.com/blog/archives/2014/09/17/how-we-write-a-gemfile)
* [Bundler and Gemfile best practices](https://depfu.com/blog/2017/01/18/bundler-and-gemfile-best-practices)
* [The Healthy Gemfile](https://content.pivotal.io/blog/the-healthy-gemfile)
* [Gemfile Best Practices & Discourse](http://mcdowall.info/gemfile-best-practices-and-discourse/)
* [Heroku: Specifying a Ruby Version](https://devcenter.heroku.com/articles/ruby-versions)
* [Bundler: Specifying a Ruby Version](http://bundler.io/v1.12/gemfile_ruby.html)
* [Kill Your Dependencies](http://www.mikeperham.com/2016/02/09/kill-your-dependencies/)

相关工具站点：

* [hakiri](https://hakiri.io/facets)
* [deppbot](https://www.deppbot.com/)

延伸阅读和相反观点，可忽略：

* [使用工具提高 Rails 代码质量](http://wjp2013.github.io/rails/refactoring-rails-wit-tool/)
* [57 Best Ruby Gems We Use at RubyGarage](https://rubygarage.org/blog/best-ruby-gems-we-use)
* [Gem Versioning and Bundler: Doing it Right](http://yehudakatz.com/2011/05/30/gem-versioning-and-bundler-doing-it-right/)
* [Bundler Best Practices](https://www.viget.com/articles/bundler-best-practices/)
