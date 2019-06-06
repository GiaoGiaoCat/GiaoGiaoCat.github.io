---
layout: post
title:  "Installing Postgres via Brew"
date:   2019-03-30 11:40:00
categories: tool
tags: mac
author: "Victor"
---

## Install

```bash
brew install postgresql
# To have launchd start postgresql now and restart at login
brew services start postgresql

# if you don't want/need a background service you can just run
pg_ctl -D /usr/local/var/postgres start
```

执行 `brew services start postgresql` 会自动在 ~/Library/LaunchAgents 下复制 homebrew.mxcl.postgresql.plist。

## Commands

```bash
# Create database
createdb <database_name>
createdb mydjangoproject_development # example

# List databases
psql -U postgres -l

# Show tables in database
psql -U postgres -d <database_name>
psql -U postgres -d mydjangoproject_development # example

# Drop database
dropdb <database_name>
dropdb mydjangoproject_development # example

# Restart database
dropdb <database_name> && createdb <database_name>
dropdb mydjangoproject_development && createdb mydjangoproject_development # example
```

## 相关链接

* [Mac 上安装配置和简单实用PostgreSQL](https://www.jianshu.com/p/9e91aa8782da)
