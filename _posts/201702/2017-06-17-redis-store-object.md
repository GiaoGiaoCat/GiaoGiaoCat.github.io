---
layout: post
title:  "Marshal.dump and load with ActiveRecord"
date:   2017-06-17 11:40:00
categories: rails
tags: tip caching
author: "Victor"
---

## Marshal

`Marshal.dump` 和 `Marshal.load` 可以把 Ruby 对象序列化为二进制数据。

## 实践

想在 Redis 中储存一个对象，并且想将来取出这个对象。

一开始想到的办法：

```ruby
# 取出对象
json_string = Redis.current.get(encrypted_id)
User.new.from_json(user_json)

# 储存对象
user = User.find(encrypted_id)
Redis.current.set(encrypted_id, user.to_json)
```

这样遇到的问题是，一旦要对 user 做一些关联性的操作，比如 `user.posts.create(xxx)` 就会提示 `user` 还没有保存。因为 `from_json` 方法确实是序列化了一个新对象出来，而不是取出原来的对象。虽然看起来各个属性值一样，但是没什么用。

正确做法：

```ruby
data = Redis.current.get(encrypted_id)
return Marshal.load(data) if data

user = User.find(encrypted_id)
Redis.current.set(encrypted_id, Marshal.dump(user))
```

PS: `JSON.parse(json_string, object_class: User)` 也是不行的 ：）

## 参考

* [Marshal](http://ruby-doc.org/core-1.9.3/Marshal.html)
* [Marshal.dump and load with ActiveRecord](https://keita.blog/2013/09/14/marshal-dump-and-load-with-activerecord/)
* [Ruby, Rails + objects serialization (Marshal), Mongoid and performance matters](https://mensfeld.pl/2014/01/ruby-rails-objects-serialization-marshal-mongoid-and-performance-matters/)
