---
layout: post
title:  "unassociated records in rails 5"
date:   2018-06-13 12:00:00

categories: rails
tags: rails5
author: "Victor"
---

```ruby
class User < ApplicationRecord
  has_one :post
end

class Post < ApplicationRecord
  belongs_to :user
end
```

posts 表中含 foreign key, 将来有需要也可以改成 `user has_many :posts`。

现在的需求是希望找到第一条 **不关联 post 的 user**。比较笨的方法是：

```bash
user = User.where("id not in (select distinct(user_id) from posts)").first
# could also be expressed as
user = User.where("id not in (?)", Post.pluck(:user_id)).first
```

其生成的 SQL 如下：

```sql
SELECT  "users".* FROM "users"
WHERE (id not in (select distinct(user_id) from posts))
ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
```

根据 [MySQL数据库开发军规](https://github.com/wjp2013/the_room_of_exercises/blob/master/guides/MySQL.md) IN 的值必须少于 50 个，所以上面的实践是有问题的。

相对来说比较好的 SQL 应该是：

```sql
select * from users
left join posts on posts.user_id = users.id
where posts.user_id is null;
```

```ruby
User.left_joins(:posts).where(posts: {user_id: nil}).first
```

## 参考链接

* [unassociated records in rails 5](https://campedersen.com/2018/05/15/unassociated/)
* [MySQL数据库开发军规](https://github.com/wjp2013/the_room_of_exercises/blob/master/guides/MySQL.md)
