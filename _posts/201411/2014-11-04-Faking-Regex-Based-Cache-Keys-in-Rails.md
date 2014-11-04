---
layout: post
title:  "Faking Regex-Based Cache Keys in Rails"
date:   2014-11-04 17:00:00
categories: rails
tags: caching
author: "Victor"
---

## 结论

这种方法在 Rails4 的时代已经可以由 **Russian Doll Caching & Cache Digests** 解决了，所以大家就是看个思路吧，别真的照着这个做。

## 正文

Rails 中有很多方法来缓存数据。比如我们可以用 Dalli 连接 Memcached 服务器当做存储器。

这个方案虽然很简单，但是有个问题。Memcache 不支持正则过滤 key 过期。这意味着你很难删除以 **user-1** 开头的，而不是 **user-2** 开头的 keys。

看起来我们只能清空全部的缓存，```Rails.cache.clear```。或者删除指定的缓存键 ```Rails.cache.delete("my-key")```。

不久前，我们有一个项目需要根据用户 ID 缓存一些数据，缓存 key 就像 **user-1-foo** 或者 **user-2-foo**

当一个用户访问了 **foo** 节点，我们为这个用户生成或取出缓存。问题是当我们指定某一个用户的缓存过期的时候，首先我们想到了 Rails 内置的正则失效机制：

```ruby
Rails.cache.delete_matched(/regex/)
```

可惜 Memcache 不支持 ```delete_matched``` 方法。这很合情理，Memcache 通过对有限且简单键值对的读和删除来加速。利用正则来删除 key 是比较复杂的，会引起性能问题，所以 Memcache 选择不支持它。

一个常见的用来规避这种问题的方法是在 key 中增加数值类型的命名空间关键字，例如 **user-1-memcache-iterator-4-foo** 和 **user-2-memcache-iterator-4-foo**

当你需要让某个用户的缓存过期的时候，只需要修改这个数值。所有的查询也都会遵循这个原则来递增。理论上原来的 key 可以删除，因为我们将来的查询再也不会访问到它。现在如果我们要刷新 user 1 的缓存， 那么我们只需要通过下面的 ```memcache_iterator``` 方法:

```ruby
class User
  def increment_memcache_iterator
    Rails.cache.write("user-#{self.id}-memcache-iterator", self.memcache_iterator + 1)
  end

  def memcache_iterator
    # fetch the user's memcache key
    # If there isn't one yet, assign it a random integer between 0 and 10
    Rails.cache.fetch("user-#{self.id}-memcache-iterator") { rand(10) }
  end
end
```

我们用另一种方法来为 **foo** 节点建立缓存 key，确保它总是使用用户现有的缓存迭代器：

```ruby
class User
  def foo_cache_key
    "user-#{self.id}-memcache-iterator-#{self.memcache_iterator}-foo"
  end
end
```

最后，我们告诉 Rails 使用此 ```foo_cache_key``` 方法时，需要从内存缓存中获取数据：

```ruby
class FooController
  caches_action :foo, :cache_path => Proc.new { |c| current_user.foo_cache_key }
end
```

该解决方案是很方便的，因为它可以让你刷新某个用户的缓存，但保留其他用户的缓存完好无损。它也极快，因为你做的就是递增缓存的整数。一个小技巧，用户的缓存迭代器被改变的那一刻，我们可以利用后台任务遍历用户来填充新的密钥缓存：

```ruby
class User
  def eager_load_foo
    if Rails.cache.fetch(self.foo_cache_key).nil? # a cache of the current iterator doesn't exist, so create it
      Rails.cache.write(self.foo_cache_key, Foo.to_json)
    end
  end
end
```

## 原文链接

* [Faking Regex-Based Cache Keys in Rails](http://quickleft.com/blog/faking-regex-based-cache-keys-in-rails)
