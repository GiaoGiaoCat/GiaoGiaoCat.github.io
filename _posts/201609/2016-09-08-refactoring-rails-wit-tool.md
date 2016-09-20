---
layout: post
title:  "使用工具提高 Rails 代码质量"
date:   2016-09-08 10:30:00
categories: rails
tags: refactoring
author: "Victor"
---

每个人都可能用正确的风格写出质量低下的代码，这其中可能包括：

* 重复的代码，它们可能存在于同一个类或不同类中
* 不一致或没有标识性的对象、变量或方法命名
* 过长的代码段
* 让人费解的布尔表达式
* 过于复杂的逻辑判断
* 对象错误地暴露其内部状态
* 遭废弃但没有删除的类或方法

对这些检查，仅靠工具可能就不够了。重构这些代码，需要我们先写测试，然后提 PR，找个人来帮我们 Review 这部分代码。

重构一个系统时，应该从引入工具开始。

## 代码风格检测
[rubocop](https://github.com/bbatsov/rubocop) 是一个 Ruby 静态代码分析工具，它基于 Ruby 社区的代码风格指南。

在 Gemfile 中增加

```ruby
group :development do
  gem 'rubocop', require: false
end
```

然后在项目的根目录增加 .rubocop.yml 配置文件。

在后面使用 `-a` 参数可以自动修复错误。

```shell
rubocop app/models/entrance_guard/key.rb -a
```

在 Atom 下配置：

```shell
gem install rubocop
apm install linter
apm install linter-rubocop
rbenv which rubocop
rubocop-auto-correct
```

去 Atom 的设置中把 rubocop-auto-correct 的自动纠正打开，再重启 Atom。

## 代码安全检测

[brakeman](https://github.com/presidentbeef/brakeman) 是一个 Ruby 静态代码安全检测工具。

在 Gemfile 中增加

```ruby
group :development do
  gem 'brakeman', require: false
end
```

brakeman 的用法还是比较简单的，平时我们在项目的目录下直接执行 `brakeman` 就行。


## 依赖安全检测

[bundle-audit](https://github.com/rubysec/bundler-audit) 检查你用的 gem 是否有安全补丁需要升级。

在 Gemfile 中增加

```ruby
group :development do
  gem 'bundler-audit', require: false
end
```

如果本地检测直接使用 `bundle-audit check --update`

## 综合代码质量审查

[Code Climate](https://codeclimate.com/) 可以检测测试覆盖率、代码复杂度、重复的代码、安全问题、代码风格等关于代码质量的问题。它集成了上面提到的全部工具。

用法还是比较简单的，团队的管理员按照说明设置一下 deploy key 就行。每次 PR 或 commit 后，Code Climate 收到 Github Webhooks 的调用开始进行统计。统计结果会在 PR 的页面或 Feed 中展现，你可以根据该结果决定是否 merge 这个 PR。

Code Climate 还提供了一个 [Chrome Extension](https://codeclimate.com/browser-extension)。装了这个插件后打开 Github 的页面，就能在浏览器里面看到 Code Climate 提供的 Code Review 和 测试覆盖率。

Code Climate 提供的命令行工具 Code Climate CLI 和 Atom Package，作用和 rubocop 几乎一样。所以可以根据喜好酌情安装。需要注意的是 Code Climate CLI 需要你安装 Docker。

Code Climate 和 Github 可以很好的结合到一起，参考[这篇文档](https://docs.codeclimate.com/docs/github)。

## CI 自动集成

[travis-ci](https://travis-ci.org/) 没啥说的，贼好。

把 rubocop 集成到 workflow 中，可以在 CI 中配置 `bundle exec rubocop`。

把 brakeman 集成到 workflow 中，可以在 CI 中配置 `bundle exec brakeman -z`。

最后把 bundler-audit 集成到 workflow 中，可以在 CI 中配置

```
bundle exec bundle-audit update
bundle exec bundle-audit -v
```

如果你已经使用了 Code Climate，就不需要在 CI 中集成它们。

## 遗留代码的重构

如果接盘了别人的代码，可以试试 [suture](https://github.com/testdouble/suture)。

## 相关链接

* [A Recipe for Rails Continuous Integration](https://mattbrictson.com/rails-continuous-integration)
* [rubycritic 中文介绍](https://ruby-china.org/topics/28746)
* [houndci](https://houndci.com/)
