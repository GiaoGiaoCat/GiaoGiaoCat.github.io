---
layout: post
title:  "Creating staging and other environments in Rails"
date:   2016-08-25 12:30:00
categories: rails
tags: environment
author: "Victor"
---

Rails 默认有三种环境 development、testing 和 production。有时我们需要一个 staging 环境。比如你在本地或者产品机器上想要运行一个和 production 略有不同的环境，大部分都一样只是数据库不同。

这时最好的办法是单独建一套 staging 环境，你需要做的是如下几步：

1. 创建一个新的 `config/environments/YOUR_ENVIRONMENT.rb` 文件
2. 在 `config/database.yml` 配置新的数据库源
3. 在 `config/secrets.yml` 添加新的 secret key

你可以直接复制 production.rb 环境文件，或者像我一样用下面的代码

```bash
$ cat config/environments/staging.rb
```

```ruby
# Just use the production settings
require File.expand_path('../production.rb', __FILE__)

Rails.application.configure do
  # Here override any defaults
  config.serve_static_files = true
end
```

```ruby
# config/database.yml
# Production settings for local development and profiling
staging:
  database: db_profile
  ..
```

```bash
$ rake secret
c975f1417b60097ecfc17e308f0d8fc502f1e2534b14ef41527d703923db9e875ad4eeb779a74c732bb6c5747c3b56d84fe7f38554089522a2f557c587766fcc
```

```ruby
# config/secrets.yml
..
test:
  secret_key_base: 40bf0f5019e785b6b44a29f1680febbcb06db8dd64f835986c6686bebddf304b67f8a9a6dffcc862f2586edc60921d0b736e3e0b1833eea2431767d2a0d1f9cc

# Add this new entry with the generated key base
staging:
  secret_key_base: c975f1417b60097ecfc17e308f0d8fc502f1e2534b14ef41527d703923db9e875ad4eeb779a74c732bb6c5747c3b56d84fe7f38554089522a2f557c587766fcc
..
```

最后不要忘了为 staging 环境创建不同的 initializers：

```bash
$ cat config/initializers/rack_profiler.rb
```

```ruby
if Rails.env.development? || Rails.env.staging?
  require 'rack-mini-profiler'

  # Initialization is skipped so trigger it
  Rack::MiniProfilerRails.initialize!(Rails.application)

  # Needed for staging env
  Rack::MiniProfiler.config.pre_authorize_cb = lambda { |env| true }
  Rack::MiniProfiler.config.authorization_mode = :allowall
end
```

没错，其他地方你也可以用 `Rails.env.staging?`，比如在 Gemfile 里面指定一些 gem 只能在 staging 环境下使用。另外也可以为指定的数据库做数据迁移。

```ruby
$ RAILS_ENV=staging rake db:create
```

### 原文

* [Creating staging and other environments in Rails](http://nts.strzibny.name/creating-staging-environments-in-rails/)
