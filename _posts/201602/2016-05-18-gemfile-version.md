---
layout: post
title:  "Gemfile 中版本号的说明"
date:   2016-05-18 10:50:00
categories: rails
tags: tip
author: "Victor"
---

```ruby
source 'https://rubygems.org'

gem 'rails', '>= 5.0.0.beta4', '< 5.1'
gem 'sqlite3'
gem 'puma', '~> 3.0'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.1.0'
gem 'jquery-rails'
gem 'turbolinks', '~> 5.x'
gem 'jbuilder', '~> 2.0'

group :development, :test do
  gem 'byebug', platform: :mri
end

group :development do
  gem 'web-console'
  gem 'listen', '~> 3.0.5'
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

### 沒加版本号

```ruby
gem 'sqlite3'
gem 'jquery-rails'
```

使用最新稳定(stable)版本。

### 加明确版本号

使用指定的版本。

### 大于，小于版本号

```ruby
gem 'uglifier', '>= 1.3.0'
gem 'rails', '>= 5.0.0.beta4', '< 5.1'
```

### 差不多

```ruby
gem 'coffee-rails', '~> 4.1.0'
```

 4.1.0 以上，但 4.2 以下(不包括 4.2)的最新版本。

## 语义化版本

版本号的三个数字分别代表主要版本号，次要版本号，修订版本号。

* 主要版本 Major：功能大改，公开的 API 做了不少修正，通常没办法向下不相容
* 次要版本 Minor：加了某些新功能，但不影响其它功能，向下相容
* 修订版本 Patch：对现有的功能做了小幅度的修正，向下相容

## 相关链接

* [Ruby 語法放大鏡之「在 Gemfile 裡看到版本寫法有好幾款，各是代表什麼意思?」](http://kaochenlong.com//2016/05/02/gemfile/)
* [Bundler](http://bundler.io/gemfile.html)
* [語意化版本 2.0.0](http://semver.org/lang/zh-TW/)
