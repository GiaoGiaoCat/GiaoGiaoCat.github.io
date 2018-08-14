---
layout: post
title:  "在单一的 Ruby 脚本里面使用 bundle 安装 gem"
date:   2018-08-13 13:00:00

categories: rails
tags: tip
author: "Victor"
---

在单文件的脚本中使用 bundle 需要在 Ruby 脚本的顶部添加 `require 'bundler/inline'`，接下来用 gemfile 的语法来声明自己的需要的 gem，下面看例子：

```ruby
require 'bundler/inline'

gemfile do
 	  source 'https://rubygems.org'
 	  gem 'json', require: false
  	  gem 'nap', require: 'rest'
  	  gem 'cocoapods', '~> 0.34.1'
end

puts 'Gems installed and loaded!'
puts "The nap gem is at version #{Nap::VERSION}"
```

运行这个脚本会自动安装缺失的 gem，例如保存这个脚本为 bundler_inline_example.rb。直接执行 `ruby bundler_inline_example.rb` 先安装缺少的 gem 再执行其中的方法。

## 相关

* [How to use Bundler in a single-file Ruby script](https://bundler.io/v1.16/guides/bundler_in_a_single_file_ruby_script.html)
