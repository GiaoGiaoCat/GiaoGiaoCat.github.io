---
layout: post
title:  "使用 create_or_find_by 避免并发问题"
date:   2018-09-20 11:00:00

categories: rails
tags: tip
author: "Victor"
---

## 问题

我们经常用 `find_or_create_by` 来处理如果记录不存在就创建一个的情况。

```bash
> User.find_or_create_by(username: "sikachu")
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."username" = ? LIMIT ?  [["username", "sikachu"], ["LIMIT", 1]]
  (0.1ms)  begin transaction
  User Create (1.9ms)  INSERT INTO "users" ("username", "created_at", "updated_at") VALUES (?, ?, ?)  [["username", "sikachu"], ["created_at", "2018-02-21 02:52:05.983257"], ["updated_at", "2018-02-21 02:52:05.983257"]]
  (0.8ms)  commit transaction
#=> #<User id: 1, username: "sikachu", created_at: "2018-02-21 02:52:05", updated_at: "2018-02-21 02:52:05">
> User.find_or_create_by(username: "sikachu")
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."username" = ? LIMIT ?  [["username", "sikachu"], ["LIMIT", 1]]
#=> #<User id: 1, username: "sikachu", created_at: "2018-02-21 02:52:05", updated_at: "2018-02-21 02:52:05">
```

在极端情况下，如果同时触发两次一样的请求，那么两次 `SELECT` 可能都找不到记录，这样就会触发两次 `INSERT` 动作。最后数据库就出现了奇怪的脏数据，两条一样的值。

## 解决

相关链接中 DHH 已经给了一个补丁来解决这个问题。使用 `create_or_find_by`，如果给定的条件不存在，更安全的方法在数据库中创建记录。该方法利用数据库的唯一性约束来避免错误流程。

```ruby
# Similar usage as the code above:
User.create_or_find_by(username: "sikachu")
# There is also a bang version which will raise if there is
# a validation error:
User.create_or_find_by!(username: "sikachu")
```

该方法内部，`create` 在意外发生时抛出一个 `ActiveRecord::RecordNotUnique` 异常，随后使用 `find_by!` 找到该记录。

虽然这违反了 [不要使用异常来进行流程控制](http://wiki.c2.com/?DontUseExceptionsForFlowControl)，但在这个情况下，应该是可行的，因为可以通过后端数据库来保持数据的一致性。

## 相关链接

* [Use create_or_find_by to avoid race condition in Rails 6.0](https://sikac.hu/use-create-or-find-by-to-avoid-race-condition-in-rails-6-0-f44fca97d16b)
* [Rails Issue#31989](https://github.com/rails/rails/pull/31989)
