---
layout: post
title:  "Guide to Caching in Rails using Memcache"
date:   2014-12-02 09:50:00
categories: rails
tags: caching
author: "Victor"
---

## Implementing and Using in Rails

### 安装 Dalli (Memcache Client)

* [Installing Memcache Dalli Client](https://github.com/mperham/dalli)
* [Dalli Client Api Doc](http://www.rubydoc.info/github/mperham/dalli/Dalli/Client)


Include ```gem 'dalli'``` in Gemfile & Run ```bundle install```

### 在 Rails 中配置 Memcache 作为缓存介质

* Set **perform_caching** and specify **cache_store** in the environment file (e.g. RAILS_ROOT/config/environments/production.rb)

```
config.action_controller.perform_caching = true
config.cache_store = :dalli_store, { :namespace => “my_project”, :expires_in => 1.day, :socket_timeout => 3, :compress => true }
```

Detailed explanation of Dalli Client options can be found [here](https://github.com/mperham/dalli)

Memcache does not provide any command to list all keys present but at times seeing the keys and their expiry time can be a life saviour. Here is a [Ruby Script to list all memcache keys](https://gist.github.com/bkimble/1365005).

### 使用 Memcache 来缓存开销较大的运算并且避免重复执行

* 我们把较大的运算结果存入 memcache 并在下次读取。如果下次计算返回相同的结果则我们直接利用缓存内容，否则将新的计算结果放入缓存中。
* Rails 提供几个辅助方法 ```Rails.cache.read``` 读取缓存，```Rails.cache.write``` 写入缓存，```Rails.cache.fetch``` 返回缓存结果，如果缓存中没有对应的值，则将新的运算结果写入缓存。

Code Snippet to pass a block to evaluate if key not present in cache, else return the value of key

```ruby
# File: application_controller.rb

# Always Calculate if caching is disabled
# Calculate the result if key not present and store in Memcache
# Return calculated result from Memcache if key is present

def data_cache(key, time=2.minutes)
  return yield if caching_disabled?
  output = Rails.cache.fetch(key, {expires_in: time}) do
    yield
  end
  return output
rescue
   # Execute the block if any error with Memcache

   return yield
end

def caching_disabled?
  ActionController::Base.perform_caching.blank?
end
```

Using ```data_cache``` helper created in last step in application_controller.rb

```ruby
# File: posts_controller.rb

respond_to :json

def index
  # Create cache to expire after 3 minutes
  all_posts_for_a_category = data_cache("#{@category.name}", 3.minutes) do
      @category.posts.all
  end
  respond_with(posts: all_posts_for_a_category)
end

# Post#index - Before Filter
def find_category
  @category = Category.where(id: params[:category_id]).first
  respond_with({message: 'Category Not Found'}, status: 404) if @category.blank?
end
```

### 清理缓存

#### Auto-Expire after a specified time

Memcache accepts an expiration time 3 option with the key to write and only returns the stored result if key has not expired yet, else returns nothing.

* ```Rails.cache.write(key, value, {expires_in: time})```
* ```Rails.cache.fetch(key, {expires_in: time}) { # Block having expensive calculations }```

#### Faking Regex based Expiry

Unfortunately, Memcache does not allow listing of all keys so, apparently, there is no way to partially expire a subset of keys. Memcache only provides a delete method for the specified key restricting us to, beforehand, know the key to delete.

But, there is a nice trick to partially expire a subset of keys [Faking Regex Based Cache keys in Rails](http://quickleft.com/blog/faking-regex-based-cache-keys-in-rails).

* To achieve it, we have to focus more on keyword subset and defining subset is the Nirvana to crack the problem.
* Introduce a dynamic component in all the keys which we want to treat as subset
* Have a way to control the change of dynamic component
* When dynamic component changes, a new key is created and stored in Memcache which would be referred for data from now.
* The old key still remains in memcache and has still not expired but it stores the stale data and fresh data is hold by new key which we are referring in all of our interactions.
* Memcache would eventually kick the old key with stale data out (uses LRU algorithm to allocate space for new data)
* In the snippet below, we have added dynamic component last_cache_refreshed_at in Category which can be selectively changed to expire cache of particular categories instead of all.

```ruby
# File: posts_controller.rb

def index
  # Update Category#last_cache_refreshed_at anytime to force the block evaulation
  # again and storing the new key, hence expiring the last key by effectively not
  # refering it for data and consulting new key for fresh data
  # Time converted to integer to avoid sanitization hassle

  all_posts_for_a_category = data_cache("#{@category.name}-#{@category.last_cache_refreshed_at.to_i}", 3.minutes) do
      @category.posts.all
  end
  respond_with(posts: all_posts_for_a_category)
end
```

## 问题

### How can I force Rails cache to not escape?

I'm writing a string to my memcached using rails (Dalli), and then using node.js (node-memcached) to read the value, and Rails is writing to memcache with these extra prepended stuff. I also checked memcache using command line.

```
# Writing with rails:
Rails.cache.write("test", 'helloworld')

# Reading from node.js:
// output
I"helloworld:ET
```

What's happening is that Dalli is calling ```Marshal.dump('helloworld')``` before writing the value to the cache. To avoid this you'll need to interact with Dalli directly instead of going through ```Rails.cache``` then you can pass the ```:raw => true``` option to make Dalli store the exact value that you pass to it.

```ruby
dcache = Dalli::Client.new
dcache.set("test", 'helloworld', 0, :raw => true)
```

## 相关阅读

* [Rails Cache Store Guide](http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html)
* [Faking Regex Based Cache keys in Rails](http://quickleft.com/blog/faking-regex-based-cache-keys-in-rails)
* [Ruby Script to list all memcache keys](https://gist.github.com/bkimble/1365005)
* [Writing Test Cases](http://vinsol.com/blog/2014/02/11/guide-to-caching-in-rails-using-memcache/#fn_3)
