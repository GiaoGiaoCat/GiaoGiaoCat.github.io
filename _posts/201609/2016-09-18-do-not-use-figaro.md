---
layout: post
title:  "不要使用 Figaro"
date:   2016-09-18 10:30:00
categories: rails
tags: tip
author: "Victor"
---

## 配置文件基础

我们在项目中总会遇到一些不希望泄漏的配置信息，比如你的短信服务商提供的 API KEY 之类的。

最简单的做法是把这些值通过环境变量的方式传递进来，比如 `API_KEY=xxxx rails s`，在项目的任何地方都可以直接 `ENV['API_KEY']`。而当环境变量多了一些的时候，就可以把这些放在 `.zshrc` 之类的地方。

另外一种做法是自己建立一个 YML 配置文件 **注意不能提交到版本库**。

```bash
echo /config/application.yml >> .gitignore
cp config/application.yml config/application.example.yml
```

```yaml
# config/application.yml
username: "admin"
password: "secret"
secret_token: "024e1460..."
```

```ruby
# config/application.rb
ENV.update YAML.load(File.read(File.expand_path('../application.yml', __FILE__)))
```

之后你就可以在项目的任何地方继续使用 `ENV[:username]` 来获取配置了。

## 根据环境获得不同的配置

后来我们需要在不同的环境中，使用不同的配置。

```yaml
# config/application.yml
username: "admin"
password: "secret"
secret_token: "024e1460..."
development:
  host: "localhost:3000"
test:
  host: "test.local"
production:
  host: "blog.example.com"
```

```ruby
# config/application.rb
CONFIG = YAML.load(File.read(File.expand_path('../application.yml', __FILE__)))
CONFIG.merge! CONFIG.fetch(Rails.env, {})
CONFIG.symbolize_keys!
```

这样，当使用 `CONFIG[:host]` 时候拿到的就是当前环境的配置了。

## 后来


后来跟大家一样，我开始使用 [figaro](https://github.com/laserlemon/figaro)。它让你不用再自己去 config/application.rb 中撸一长串代码，省了不少事。

## 为什么不要使用 Figaro

盲目的使用了一年之后，我发现这玩意和 Rails 4 引入的 secrets.yml 文件相比没差多少。你可以看一下 Figaro 在 Github 上的 README，它有一份很详细的对比。

那干吗还有装一个 gem，在 config/application.rb 文件的最下面增加一行 `CONFIG = Rails.application.secrets`，以后用到配置的地方直接写 `CONFIG.hello` 或者 `CONFIG.name["first"]` 好了。

只要注意什么时候用访问方法，什么时候用 hash 取值。另外 secrets.yml 文件也不要纳入版本控制才对。

```yaml
development:
  secret_key_base: 024e1460...
  hello: hehe
  name:
    first: victor

test:
  secret_key_base: 024e1460...

production:
  secret_key_base: 024e1460...
```
