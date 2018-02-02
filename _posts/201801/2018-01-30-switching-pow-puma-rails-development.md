---
layout: post
title:  "从 Pow 切换到 Puma-Dev"
date:   2018-01-30 17:10:00

categories: tool
tags: terminal
author: "Victor"
---

本文的前提是你已经用了 pow，但是没有用 `Powprox 或 Invoker` 捣鼓过本地开发环境的 HTTPS。

## 起因

Pow 不支持 HTTPS，而这几乎是近年 Web 的标配之一了。我想在本地的开发环境也捣鼓捣鼓这玩意。

## 安装

1. Uninstall Pow
  * 根据你安装 pow 方式的不同，使用 `curl get.pow.cx/uninstall.sh | sh` 或者 `brew uninstall pow`
2. Add Puma to your Rails app
  * Gemfile 中添加 `gem 'puma'`
  * `bundle install`
3. Install puma-dev
  * `brew install puma/puma/puma-dev` 安装
  * `sudo puma-dev -setup` 自动配置 DNS 并生成开发使用的 CA 证书
4.
  * `puma-dev -install` 后台监听本地 80 和 443 端口以便处理 `.dev` 域名的请求

## 使用

### 链接一个项目

```bash
cd ~/projects/my-project
puma-dev link
```

这样会将 `my-project.dev` 链接到 `~/projects/my-project`

### 用别名链接一个项目

`puma-dev link -n other-name ~/projects/my-project`

### 域名

详细内容在 [Chrome 63 forces .dev domains to HTTPS via preloaded HSTS](https://ma.ttias.be/chrome-force-dev-domains-https-via-preloaded-hsts/) 这里有介绍。

太长不想读，不想听原理，就说怎么办吧。

如果站点没用 HTTPS，那不用 `site.dev` 改用 `site.localhost` 就行，什么配置都不需要做。

### 环境变量

* 首先加载 `source ~/.powconfig`
* 然后利用 `source` 命令加载项目目录下的 `.env, .powrc, .powenv`
* 如果以前使用 pow 的话，注意 `POW_*` 开头的变量都无效了
* puma-dev 提供 `CONFIG, THREADS, WORKERS` 方便你控制 puma 的启动

### Webpacker 配置

1. `bundle binstub bundler --force`
2. 参考 [webpack-dev-server.md](https://github.com/rails/webpacker/blob/master/docs/webpack-dev-server.md)
  * `config/webpacker.yml` 设置 https 参数为 true
  * 按照文档解决 `https://localhost:3035/sockjs-node/info?t=1503127986584 net::ERR_INSECURE_RESPONSE` 报错的问题
3. 新开一个 terminal 窗口运行 `./bin/webpack-dev-server`

## 最后

并没有简单多少，你需要开至少 4 个窗口。

1. `./bin/webpack-dev-server` 加速你的前端编译
2. `tail -f log/development.log` 查看开发日志
3. 随时 `rails restart`
4. 要看一下 puma-dev 日志 `tail -f ~/Library/Logs/puma-dev.log`

## 其它

### 常用命令

* `tail -f ~/Library/Logs/puma-dev.log` 查看日志
* `puma-dev -stop` 关闭
* `puma-dev -install` 启动
* `curl -H "Host: puma-dev" localhost/status` 查看状态

### SSL 证书无效

1. 关闭 SIP，参考 [Unix/Linux 系统中的 Operation Not Permitted 问题](http://www.barretlee.com/blog/2016/04/06/operation-not-permitted-problem-in-linux-or-unix-system/)
2. 将证书移动到 System 目录下，参考原文中关于 Invalid SSL certificate 的部分

### DNS 未生效

1. 重连 WiFi
2. 重启浏览器

## 参考

* [Puma-dev](https://github.com/puma/puma-dev)
* [原文](https://stormconsultancy.co.uk/blog/development/ruby-on-rails/switching-pow-puma-rails-development/)
* [Puma-dev support Rails' webpack](https://github.com/puma/puma-dev/issues/123)
* [用 Puma-dev 作為 Rails 開發環境伺服器](https://icook.engineering/%E7%94%A8-puma-dev-%E4%BD%9C%E7%82%BA-rails-%E9%96%8B%E7%99%BC%E4%BC%BA%E6%9C%8D%E5%99%A8-341be0add9f0)
