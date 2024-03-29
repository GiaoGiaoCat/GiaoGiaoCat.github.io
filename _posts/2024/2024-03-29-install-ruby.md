---
layout: post
title:  "新电脑安装 Ruby"
date:   2024-03-28 08:10:00

categories: ruby
tags: terminal
author: "Victor"
---

```shell
brew install rbenv
```

当您在使用 rbenv 时运行 `rbenv version` 命令，并且返回 "system" 时，表示您当前处于系统（system）Ruby 环境下。

系统 Ruby 环境是指 macOS 默认安装的 Ruby 版本，通常是为了系统级任务而安装的。在系统 Ruby 环境中，您可能无法直接使用 rbenv 来管理 Ruby 版本。相反，系统 Ruby 通常用于操作系统本身和其他系统级任务。

如果您想要在 rbenv 环境下安装和管理不同版本的 Ruby，您可以按照之前提到的步骤安装和设置 rbenv，并且运行 rbenv versions 来查看您当前已安装的 Ruby 版本，并使用 rbenv global 来设置全局版本。


```shell
CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl)" rbenv install 3.3.0
rbenv global 3.3.0
rbenv rehash
ruby -v
```

在你需要更新 gem 的仓库下执行

```shell
bundle update --bundler
bundle update
```

今天在更新 jeklly 的时候遇到问题，最后的解决方案是删掉 Gemfile.lock 重新执行 `bundle install` 并通过 `bundle exec jekyll serve` 测试是否可以本地运行。
