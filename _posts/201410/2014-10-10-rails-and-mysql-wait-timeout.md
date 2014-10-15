---
layout: post
title:  "Using rails and lost connection to MySQL server during queryk"
date:   2014-10-15 17:20:00
categories: rails
tags: tip
author: "Victor"
---

## 一句话结论

**mysql2 gem** 设置了一个特殊的默认值 ```wait_timeout``` 为 2147483 秒。它会覆盖你在 mysql 服务器上的设置。

## 经过

我在 mysql 服务器上设置了 [wait_timeout](http://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_wait_timeout) 参数，但是不生效。

起因是我们的 mysql 服务器出现了内存不足的情况：

* 它比提供同样服务的其它几台服务器内存更少
* 它产生过多的连接 (12k) 左右，并且因为没有设置交换内存，而占用了很多的物理内存，最终被 OS 给杀死

你可能觉得，只要给服务器升级内存就好了啊。但是因为某些原因我不能给这个机器升级，只能找到问题的原因换个方法解决，因为有其它的机器和这台配置一样，但是就没问题。并且那台机器的数据库连接数是 8k 左右。

通过检查 mysql 的进程，发现 80% 左右的进程都长时间处在 sleep 状态，所以只要给 ```wait_timeout``` 设置一个较小的值，比如 **120** 秒，应该可以解决这个问题。唯一的麻烦是，可能引起另外一个错误 ```mysql server has gone away```。哈哈，不管了，先试试吧。

首先在 console 里面修改 mysql 的配置，结果不好使。然后我又手动编辑 **my.cnf** 文件，再重启 mysql 进程，慢慢等待我 11G 的 mysql 数据在内存里面刷新......然后，还是老样子。不！好！使！

然后google到这个 [mysql2 gem does this thing ](https://github.com/rails/rails/issues/6441)

* [ConnectionPool](https://github.com/rails/rails/blob/3-2-stable/activerecord/lib/active_record/connection_adapters/abstract/connection_pool.rb) uses wait_timeout to decide how long to wait for a connection checkout on a full connection pool, and it defaults to 5 seconds.
* [mysql2_adapter](https://github.com/rails/rails/blob/3-2-stable/activerecord/lib/active_record/connection_adapters/mysql2_adapter.rb) uses this same wait_timeout , but passes it directly to mysql, and defaults to a much larger 2592000!!

**mysql2 gem** 设置了一个默认的 ```wait_timeout``` 值为 25 天(2147483秒)。它会覆盖掉 mysql 服务器的设置。

在 ```config/database.yml``` 中设置 ```wait_timeout``` 参数。当 mysql 连接的 sleep 时长超过这个值会被杀掉连接。并会引起 **mysql server has gone away** 错误。但是你可以设置 ```reconnect: true``` 来修复这个问题。

具体的过程是：当 active record 执行查询的时候，mysql 连接会被重复使用，且时间被重置。但是之后，会使用刚才 mysql 的配置文件中的 ```wait_timeout``` 值来进行判断。

所以，如果你在 ```config/database.yml``` 中设置超时时间是 10 秒，然后在 ```my.cnf``` 中设置超时时间 15秒。那么当你第一次执行 active record 操作的之后，如果 sleep 在 10 秒之内，又执行了一次 active record 操作。那么该命令会执行并且 mysql 连接再次进入 sleep 状态，不过这之后，sleep 超过 15 秒就会被关闭。

### 相关链接

* [Testing mysql2 wait_timeout param](https://gist.github.com/flackou/1514154)
* [wait_timeout configuration options in Rails 4](https://github.com/rails/rails/blob/v4.0.0/activerecord/lib/active_record/connection_adapters/abstract_mysql_adapter.rb#L752)
* [Using rails and having trouble getting Mysql’s wait_timeout to work? read this.](http://www.concept47.com/austin_web_developer_blog/mysql/using-rails-and-having-trouble-setting-mysqls-wait_timeout-read-this/)
