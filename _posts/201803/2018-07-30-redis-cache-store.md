---
layout: post
title:  "Rails 5.2 的 Redis Cache Store"
date:   2018-07-30 13:00:00

categories: rails
tags: caching
author: "Victor"
---

过去想在 Rails 中使用 redis 当作 cache store 的话需要使用 redis-rails gem。而 Rails 5.2 之后已经内置类 redis cache store 类型，可以用来处理 fragment cache。

需要注意的是内置的 redis cache store 无法处理 Session 和 HTTP Cache，所以说这一点是不如 redis-rails 的。

抛弃 redis-rails 的另外一个原因是，Rails 5.2 支持 `write_multi` 只有使用内置的 cache store 才能很好的使用新方法。

## 功能

* Supports vanilla Redis, hiredis, and Redis::Distributed.
* Supports Memcached-like sharding across Redises with Redis::Distributed.
* Fault tolerant. If the Redis server is unavailable, no exceptions are raised. Cache fetches are treated as misses and writes are dropped.
* Local cache. Hot in-memory primary cache within block/middleware scope.
* read_/write_multi support for Redis mget/mset. Use Redis::Distributed 4.0.1+ for distributed mget support.
* delete_matched support for Redis KEYS globs.

## 使用

```ruby
gem 'redis'
gem 'hiredis', '~> 0.6.1'
```

不需要配置 Redis cache store 会自动尝试使用更快的 hiredis。

```ruby
# config/environments/*.rb
config.cache_store = :redis_cache_store, { url: ENV['REDIS_URL'] }
```

产品环境下可能配置会比较复杂

```ruby
cache_servers = %w(redis://cache-01:6379/0 redis://cache-02:6379/0)
config.cache_store = :redis_cache_store, { url: cache_servers,

  connect_timeout:    30,  # Defaults to 20 seconds
  read_timeout:       0.2, # Defaults to 1 second
  write_timeout:      0.2, # Defaults to 1 second
  reconnect_attempts: 1,   # Defaults to 0

  error_handler: -> (method:, returning:, exception:) {
    # Report errors to Sentry as warnings
    Raven.capture_exception exception, level: 'warning',
      tags: { method: method, returning: returning }
  }
}
```

ActiveSupport::Cache::Store 本身就支持的参数仍然有效。

* `:namespace` - This option can be used to create a namespace within the cache store. It is especially useful if your application shares a cache with other applications.
* `:compress` - Enabled by default. Compresses cache entries so more data can be stored in the same memory footprint, leading to fewer cache evictions and higher hit rates.
* `:compress_threshold` - Defaults to 1kB. Cache entries larger than this threshold, specified in bytes, are compressed.
* `:expires_in` - This option sets an expiration time in seconds for the cache entry when it will be automatically removed from the cache.
* `:race_condition_ttl` - This option is used in conjunction with the `:expires_in` option. It will prevent race conditions when cache entries expire by preventing multiple processes from simultaneously regenerating the same entry (also known as the dog pile effect). This option sets the number of seconds that an expired entry can be reused while a new value is being regenerated. It's a good practice to set this value if you use the `:expires_in` option.

[详见这里](https://api.rubyonrails.org/classes/ActiveSupport/Cache.html)

同样也可以配置多个 Redis Server，注意下面的 `driver: :hiredis` 参数是多余的。

```ruby
redis_servers = %w[
  redis://localhost:6379/0
  redis://localhost:6380/0
  redis://localhost:6381/0
]

config.cache_store = :redis_cache_store, { driver: :hiredis, url: redis_servers }
```

## 一些注意事项

**redis 和 memcached 一样都会在满了之后自动回收旧对象空间。**

* 默认情况下，Redis 不会给缓存的键设置过期时间，所以尽量使用专用的 Redis 缓存服务器，绝对不要使用持久化的 Redis 服务器用来存储缓存。
* 你可以在缓存专用的 redis 服务器上更改 maxmemory-policy 参数，以便当内存不足时根据策略来清理缓存数据。
* 将缓存的读写超时时间设置的相对低一些。重新生成缓存值通常比等待超过一秒钟来检索它要快。读取和写入超时都默认为1秒，但是如果网络持续低延迟，也可以设置得更低。
* 默认情况下，如果请求期间连接失败，缓存存储不会尝试重新连接到Redis。如果你经常断开连接，也可以进行配置启用重新连接。

### 处理异常

缓存读取和写入从不引发异常，只会返回 `nil` 看起来就像没有缓存。为了发现缓存的异常，可以使用 `error_handler` 来向异常收集服务发送报告，比如 Log 或者 newrelic。

该方法接收3个关键字参数：`method` 最初调用缓存方法的方法名；`returning` 返回给用户的值，通常为 `nil`；`exception` 捕获的异常。


## 相关链接

* [Caching with Rails: An Overview](https://edgeguides.rubyonrails.org/caching_with_rails.html)
* [Built-in Redis cache store](https://github.com/rails/rails/pull/31134)
* [Class: ActiveSupport::Cache::RedisStore](https://www.rubydoc.info/gems/redis-activesupport/4.1.5/ActiveSupport/Cache/RedisStore)
