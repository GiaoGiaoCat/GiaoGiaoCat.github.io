---
layout: post
title:  "控制台的一些小技巧"
date:   2018-08-30 14:00:00

categories: rails
tags: debug
author: "Victor"
---

### 结束后回滚

```ruby
$ rails console --sandbox
```

沙箱模式下的所有 commit 都在一个事务中，所以在退出的时候会回滚所有数据操作。

### 检索上一次执行结果

通过 `_` 获取控制台上一次执行结果。

```ruby
>> Game.all.map(&:name)
=> ["zelda", "mario", "gta"]
>> names = _
=> ["zelda", "mario", "gta"]
```

### 通过 grep 过滤方法

```ruby
>> Game.first.methods.grep(/lumn/)
Game Load (0.8ms)  SELECT  "games".* FROM "games" ORDER BY "games"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> [:column_for_attribute, :update_column, :update_columns]
```

### 找到方法的位置

```ruby
>> 'Luis Vasconcellos'.method(:inquiry).source_location
=> ["/usr/local/bundle/gems/activesupport-5.2.1/lib/active_support/core_ext/string/inquiry.rb", 12]
```

### 将该方法的源代码输出到控制台

```ruby
>> 'Luis Vasconcellos'.method(:inquiry).source.display
def inquiry
    ActiveSupport::StringInquirer.new(self)
  end
=> nil
```

### helper 对象

通过 helper 对象可以直接调用 Rails 的视图 helper 方法。

```ruby
>> helper.truncate('Luis Vasconcellos', length: 9)
=> "Luis V..."
```

### app 对象

app 基本上就是应用程序的实例。通过它可以实现一些有趣的用法。

简单的比如 get 和 post：

```bash
>> app.get('/')
Started GET "/" for 127.0.0.1 at 2018-08-25 22:46:52 +0000
   (0.5ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Processing by HomeController#show as HTML
  Rendering home/show.html.erb within layouts/application
  Rendered home/show.html.erb within layouts/application (11417.2ms)
  Rendered shared/_menu.html.erb (3.6ms)
  Rendered shared/header/_autocomplete.html.erb (292.2ms)
  Rendered shared/_header.html.erb (312.9ms)
  Rendered shared/_footer.html.erb (3.7ms)
Completed 200 OK in 11957ms (Views: 11945.5ms | ActiveRecord: 0.0ms)
=> 200
```

```bash
>> app.post('/games/zelda/wishlist_placements.js')
Started POST "/games/zelda/wishlist_placements.js" for 127.0.0.1 at 2018-08-25 23:03:21 +0000
Processing by OwnlistPlacementsController#create as JS
  Parameters: {"game_slug"=>"zelda"}
  Game Load (0.6ms)  SELECT  "games".* FROM "games" WHERE "games"."slug" = $1 LIMIT $2  [["slug", "zelda"], ["LIMIT", 1]]
  Rendering wishlist_placements/create.js.erb
  Rendered wishlist_placements/create.js.erb (194.8ms)
Completed 200 OK in 261ms (Views: 252.9ms | ActiveRecord: 0.6ms)
=> 200
```

复杂一些搜索和 game 相关的 routes：

```bash
>> app.methods.grep(/_path/).grep(/game/)
=> [:search_games_path, :game_ownlist_placements_path, :game_ownlist_placement_path, :game_wishlist_placements_path, :game_wishlist_placement_path, :game_path]
```

还能结合在一起用。

```bash
>> app.get(app.root_path)
Started GET "/" for 127.0.0.1 at 2018-08-26 02:27:40 +0000
Processing by HomeController#show as HTML
  Rendering home/show.html.erb within layouts/application
  Rendered home/show.html.erb within layouts/application (12550.2ms)
  Rendered shared/_menu.html.erb (3.8ms)
  Rendered shared/header/_autocomplete.html.erb (1.2ms)
  Rendered shared/_header.html.erb (28.0ms)
  Rendered shared/_footer.html.erb (3.8ms)
Completed 200 OK in 12835ms (Views: 12810.0ms | ActiveRecord: 0.0ms)
=> 200

>> app.body.response
=> "\n<!DOCTYPE html>\n<html>\n  <head>\n    <title> ...

>> app.cookies
=> #<Rack::Test::CookieJar:0x0000556ee95c33e0 @default_host="www.example.com", @cookies=[#<Rack::Test::Cookie:0x0000556eeb72b2d0 @default_host="www.example.com", ...
```

## 原文

* [Rails Console Magic Tricks](https://medium.com/@lfv89/rails-console-magic-tricks-da1fdd657d32)
