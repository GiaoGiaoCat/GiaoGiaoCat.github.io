---
layout: post
title:  "Memcached"
date:   2014-10-31 11:40:00
categories: tool
tags: mac
author: "Victor"
---

## 介绍

* **Memcached** 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。
* 它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。
* **Memcached** 基于一个存储键/值对的hashmap。其守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。
* **Memcached** 缺乏认证以及安全管制，这代表应该将 **Memcached** 服务器放置在防火墙后。
* 一般的使用目的是，通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。
* 为了提高性能，**Memcached** 中保存的数据都存储在 **Memcached** 内置的内存存储空间中。由于数据仅存在于内存中，因此重启 **Memcached** 、重启操作系统会导致全部数据消失。另外，内容容量达到指定值之后，就基于LRULeast Recently Used)算法自动删除不使用的缓存。
* **Memcached** 本身是为缓存而设计的服务器，因此并没有过多考虑数据的永久性问题。

## Install

```bash
brew install memcached
# To have launchd start memcached at login:
ln -sfv /usr/local/opt/memcached/*.plist ~/Library/LaunchAgents
# Then to load memcached now:
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.memcached.plist
```

## Memcache Telnet Interface

### How To Connect

```bash
ps -ef | grep memcached
telnet localhost 11211
```

返回

```
Trying ::1...
Connected to localhost.
Escape character is '^]'.
```

现在可以输入一些 memcached 的命令进行测试了。

如要退出 telnet 可以输入 ```ctrl + ]``` 输入 ```?``` 查看帮助。


## 命令详解

### 存储命令

存储命令的格式：

```
<command name> <key> <flags> <exptime> <bytes>
<data block>
```

参数说明如下：

command name | **set/add/replace**
--- | ---
key | 查找关键字
flags | 客户机使用它存储关于键值对的额外信息
exptime | 该数据的存活时间，0表示永远
bytes | 存储字节数
data block | 存储的数据块（可直接理解为key-value结构中的value）

#### 添加

* ```set``` 命令不但可以简单添加，如果 ```set``` 的 ```key``` 已经存在，该命令可以更新该 ```key``` 所对应的原来的数据，也就是实现更新的作用。
* 可以通过 ```get``` 键名的方式查看添加进去的记录。
* 以通过 ```delete``` 命令删除掉，然后重新添加。
* 只有数据不存在时进行添加的 ```add```。
* 只有数据存在时进行替换的 ```replace```。

```
set username 0 0 6
victor
STORE

get username
VALUE username 0 6
victor
END

set username 0 0 4
wang
STORE

get username
VALUE username 0 4
wang
END
```

```
add useranme 0 0 6
victor
STORE

add useranme 0 0 6
victor
NOT_STORE
```

```
replace content 0 0 5
hello
NOT_STORE

set content 0 0 5
hello
STORE

replace content 0 0 2
hi
STORE

get content
VALUE content 0 0 2
hi
END
```

#### 删除

删除已存在的键值和不存在的记录可以返回不同的结果。

```
delete username
DELETE

delete useranme
NOT_FOUND
```

### 读取命令

* ```get``` 命令的 key 可以表示一个或者多个键，键之间以空格隔开。
* ```gets``` 命令比普通的 ```get``` 命令多返回了一个数。这个数字可以检查数据是否发生改变。当 key 对应的数据改变时，这个多返回的数字也会改变。
* ```cas``` 即 checked and set 的意思，只有当最后一个参数和 ```gets``` 所获取的参数匹配时才能存储，否则返回 ```EXISTS```。

```
get username content
VALUE username 0 6
victor
VALUE content 0 5
hello
END
```

```
get username
VALUE username 0 6
victor
END

gets username
VALUE username 0 6 13
victor
END

replace username 0 0 4
wang
STORE

gets username
VALUE username 0 4 15
wang
END
```

```
gets username
VALUE username 0 4 15
wang
END

cas username 0 0 6 12
victor
EXISTS

cas username 0 0 6 15
victor
STORE

get useranme
username 0 0 6
victor
END
```

### 其他常见命令

* ```append``` 在现有的缓存数据后添加缓存数据，如现有缓存的 key 不存在服务器响应为 ```NOT_STORED```。
* ```prepend``` 和 ```append``` 非常类似，但它的作用是在现有的缓存数据前添加缓存数据。
* 对于存储为数字型的可以通过 ```incr/decr``` 命令进行增减操作。
* ```flush_all``` 可指定一个参数(缺省是当前)，它的作用可以认为是让缓存全部清空。

```
get username
username 0 0 6
victor
END

append username 0 0 5
 wang
STORE

get username
username 0 0 11
victor wang
END
```

```
get username
username 0 0 6
victor
END

flush_all
OK

get username
END
```


### Supported Commands


| Command | Description | Example |
| --- | --- | --- |
| get | Reads a value | get mykey |
| set | Set a key unconditionally | set mykey 0 60 5 |
| add | Add a new key | add newkey 0 60 5 |
| replace | Overwrite existing key | replace key 0 60 5 |
| append | Append data to existing key | append key 0 60 15 |
| prepend | Prepend data to existing key | prepend key 0 60 15 |
| incr | Increments numerical key value by given number | incr mykey 2 |
| decr | Decrements numerical key value by given number | decr mykey 5 |
| delete | Deletes an existing key | delete mykey |
| flush_all | Invalidate specific items immediately | flush_all |
| flush_all | Invalidate all items in n seconds | flush_all 900 |
| stats | Prints general statistics | stats |
| stats | Prints memory statistics | stats slabs |
| stats | Prints memory statistics | stats malloc |
| stats | Print higher level allocation statistics | stats items |
| stats |  | stats detail |
| stats |  | stats sizes |
| stats | Resets statistics | stats reset |
| version | Prints server version. | version |
| verbosity | Increases log level | verbosity |
| quit | Terminate telnet session | quit |

### Traffic Statistics

你可以使用命令 ```stats``` 来查询现在的流量统计，该命令会返回目前服务器连接的进出数据字节数，结果如下：

```
STAT pid 14868
STAT uptime 175931
STAT time 1220540125
STAT version 1.2.2
STAT pointer_size 32
STAT rusage_user 620.299700
STAT rusage_system 1545.703017
STAT curr_items 228
STAT total_items 779
STAT bytes 15525
STAT curr_connections 92
STAT total_connections 1740
STAT connection_structures 165
STAT cmd_get 7411
STAT cmd_set 28445156
STAT get_hits 5183
STAT get_misses 2228
STAT evictions 0
STAT bytes_read 2112768087
STAT bytes_written 1000038245
STAT limit_maxbytes 52428800
STAT threads 1
END
```

### Memory Statistics

你可以使用命令 ```stats slabs``` 来查询现在的内存使用状况，结果如下：

```
STAT 1:chunk_size 80
STAT 1:chunks_per_page 13107
STAT 1:total_pages 1
STAT 1:total_chunks 13107
STAT 1:used_chunks 13106
STAT 1:free_chunks 1
STAT 1:free_chunks_end 12886
STAT 2:chunk_size 100
STAT 2:chunks_per_page 10485
STAT 2:total_pages 1
STAT 2:total_chunks 10485
STAT 2:used_chunks 10484
STAT 2:free_chunks 1
STAT 2:free_chunks_end 10477
[...]
STAT active_slabs 3
STAT total_malloced 3145436
END
```

If you are unsure if you have enough memory for your memcached instance always look out for the "evictions" counters given by the "stats" command. If you have enough memory for the instance the "evictions" counter should be 0 or at least not increasing。

### Which Keys Are Used?

没有内置命令来显示当前正在被设置的键，但是我们至少可以使用 ```stats items``` 来查看都有哪些键值存在：

```
stats items
STAT items:1:number 220
STAT items:1:age 83095
STAT items:2:number 7
STAT items:2:age 1405
[...]
END
```

注意这里的 **STAT items** 面的第一个数字，它代表 **STAT items** 的 id。

利用 ```stats cachedump slab_id limit_num``` 来查看, **limit_num** 表示返回多少条记录，如果设置为 0 则返回所有记录。

```
stats cachedump 1 0
ITEM username [8 b; 1320561428 s]
ITEM content [8 b; 1320561428 s]
ITEM title [7 b; 1320561428 s]
END
```

通过 ```stats items```、```stats cachedump slab_id limit_num``` 配合 ```get``` 命令可以遍历 **Memcached** 的记录。

## 相关链接

* [Memcache Telnet Interface](http://lzone.de/articles/memcached.htm)
* [Memcached: List all keys](http://www.darkcoding.net/software/memcached-list-all-keys/)
* [Memcached](http://memcached.org)
* [Memcached常用命令及使用说明](http://www.cnblogs.com/jeffwongishandsome/archive/2011/11/06/2238265.html)
* [MemAdmin](http://www.junopen.com/memadmin/)
