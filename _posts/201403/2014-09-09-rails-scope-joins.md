---
layout: post
title:  "ActiveRecord::QueryMethods 中 joins 的一种用法"
date:   2014-09-09 16:20:00
categories: rails
tags: tip
author: "Victor"
---

要根据一个 role，找到所有 user。

```ruby
class User < ActiveRecord::Base
  has_many :members
  has_many :roles, through: :members

  scope :by_role, ->(role) { joins(:roles).where( roles: { id: role }) }
end

class Member < ActiveRecord::Base
  belongs_to :user
  has_many :member_roles
  has_many :roles, through: :member_roles
end

class MemberRole < ActiveRecord::Base
  belongs_to :role
  belongs_to :member
end

class Role < ActiveRecord::Base
end
```

查询可以这样

```ruby
User.by_role(2).to_sql
```

生成的 SQL

```sql
"SELECT "users".* FROM "users" INNER JOIN "members" ON "members"."user_id" = "users"."id" INNER JOIN "member_roles" ON "member_roles"."member_id" = "members"."id" INNER JOIN "roles" ON "roles"."id" = "member_roles"."role_id" WHERE "roles"."id" = 2"
```

也可以改成

```ruby
class User < ActiveRecord::Base
  has_many :members

  scope :by_role, ->(role) { joins(members: :roles).where( roles: { id: role }) }
end
```
