---
layout: post
title:  "preload, eager_load, includes 和 joins 的区别"
date:   2019-02-14 12:00:00

categories: rails
tags: tip
author: "Victor"
---

首先直接看例子来理解它们的不同。

### Preload

它通过两次查询装载数据。

```ruby
Space.preload(:topics)
Space Load (60.3ms) SELECT `spaces`.* FROM `spaces` WHERE `spaces`.`deleted_at` IS NULL
Topic Load (179.3ms) SELECT `topics`.* FROM `topics` WHERE `topics`.`deleted_at` IS NULL AND `topics`.`space_id` IN (3, 4)
```

它会默认包含这两条查询关联的数据。

但是我们不能使用 preload 为关联表添加查询条件。当我们在关联表上执行 where 或 order 的时候就会报错。

```ruby
Space.preload(:topics).where("topics.id = 'sssssssssss'")
Unknown column 'topics.name' in 'where clause'
```

### eager_load

通过一次查询装载所有数据。它使用 left outer join 来组装数据，所以我们可以使用 where 来过滤关联表的数据

```ruby
Space.eager_load(:topics).to_a
SELECT `spaces`.`id` AS t0_r0, `spaces`.`name` AS t0_r1, `spaces`.`description` AS t0_r2, ..., `topics`.`id` AS t1_r0, `topics`.`space_id` AS t1_r1, `topics`.`author_id` AS t1_r2, `topics`.`body` AS t1_r3 FROM `spaces` LEFT OUTER JOIN `topics` ON `topics`.`space_id` = `spaces`.`id` AND `topics`.`deleted_at` IS NULL WHERE `spaces`.`deleted_at` IS NULL
```

### Joins

通过一次查询装载所有数据，不同之处是使用 inner join 来取数据。

```ruby
Space.joins(:topics).where(:topics => { body: 'sss' }).to_a
Space Load (25.2ms)  SELECT `spaces`.* FROM `spaces` INNER JOIN `topics` ON `topics`.`space_id` = `spaces`.`id` AND `topics`.`deleted_at` IS NULL WHERE `spaces`.`deleted_at` IS NULL AND `topics`.`body` = 'sss'
```

使用 joins 捞出来的数据，我们可以很方便的对关联表的数据进行过滤，但如果我们想展示关联表的数据，它会为每一列数据加载单独的查询。为了防止 N+1 查询，可以改用 includes 来加载关联表的数据。

### Includes

它很智能，默认情况下它的行为和 preload 一致，但如果我们在关联表上做一些查询和过滤，它又会自动变成 eager_load 的行为。

## 原文

* [Preload, Eager Load, Includes and Joins in Ruby on Rails](https://www.railscarma.com/blog/technical-articles/preload-eager-load-includes-and-joins-in-ruby-on-rails/)
