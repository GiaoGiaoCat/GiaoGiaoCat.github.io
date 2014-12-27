---
layout: post
title:  "Ruby 2.1.5 标准库"
date:   2014-12-25 14:30:00
categories: ruby
tags: tip
author: "Victor"
---

### 文本

名称 | 作用
--- | ---
abbrev | 计算一组字符串的可能缩写
base64 | 处理Base64编码的模块
csv | CSV文件读取和写入
digest | 消息摘要库
erb | 嵌入Ruby
json | JSON文件读取和写入
psych | YAML文件的解析
yaml | 序列化YAML格式的文件
rexml | XML文件的读取和写入
rss | 产出 RSS feeds
securerandom | 生成安全随机数
shellwords | 根据UNIX Bourne Shell的规则解析字符串
stringio | 将字符串伪装成IO
strscan | 高速Scanner
zlib | 字符串压缩

### 文件

名称 | 作用
--- | ---
coverage | 为Ruby提供覆盖测量功能(实现性质)
fileutils | 操作utility(ftools.rb 的升级版)
find | 文件搜索模块
io/console | 提供了扩展方法与控制台交互
io/nonblock | ?
io/wait | ?
pathname | 路径名类
tempfile | 生成临时文件
tmpdir | 返回临时目录
un | 类似 Unix 命令的文件操作 utility

### 网络

名称 | 作用
--- | ---
cgi | 包含了 CGI 类。用于检索 HTTP 请求参数，管理 cookies，生成 HTML 输出
drb | dRuby 是 Ruby 的分布式对象系统
gserver | GServer 实现了通用服务器，具有线程池的管理，简单的日志记录和多服务器管理
ipaddr.rb | IP地址类(IPAddr)
net/ftp | 网络
net/http | 网络
net/imap | 网络
net/pop | 网络
net/smtp | 网络
net/telnet | 网络
open-uri | open() 的 URI 支持扩展
openssl | Ruby/OpenSSL
resolv | Ruby 版 Resolver
resolv-replace | 在处理 Socket 相关类名时使用
rinda | 分布式计算，是 drb 的一部分
socket | socket 扩展库
webrick | WEB server toolkit
xmlrpc | XMLRPC 是一个轻量级协议，它允许通过HTTP远程过程调用

### 输入输出

名称 | 作用
--- | ---
open3 | access to stdin, stdout, stderr and a thread to wait for the child process when running another program.
scanf | 实现 C 的 scanf 函数

### 日语

名称 | 作用
--- | ---
nkf | 日语字符代码编码转换

### 数学

名称 | 作用
--- | ---
bigdecimal | 任意精度的浮点数字算术
cmath | 提供三角函数，复数的超越函数
mathn | 提供数学运算比如四舍五入，幂运算
matrix | 矩阵类
prime | 质数
set | 无序集合

### 数据库

名称 | 作用
--- | ---
dbm | provides a wrapper to a Unix-style dbm or Database Manager library(可将ndbm用作哈希表的库)
gdbm | 将 gdbm (GNU dbm)用作哈希表的库
pstore | 实现基于哈希基于文件的持久性机制，既对象持久化
sdbm | SDBM 提供了一个简单的基于文件的 key-value 存储，它只能存储字符串格式的键和值

### 画面控制/CUI

名称 | 作用
--- | ---
expect | 在脚本中控制交互程序
fcntl | 该模块中囊括了fcntl(2)中用到的常数
irb | Interactive Ruby (Ruby 的交互界面)
pty | 处理伪终端(Pseudo tTY)的模块
readline | GNU Readline 接口
shell | UNIX shell 命令接口

### GUI

名称 | 作用
--- | ---
tk | Ruby/Tk

### 日期·时间

名称 | 作用
--- | ---
date | 日期类
time | 字符串和 Time 对象的变换

### 多线程·同步

名称 | 作用
--- | ---
monitor | 为互斥块锁定对象
```mutex_m``` | 任何对象扩展或包括 ```Mutex_m``` 将被视为一个互斥
sync | ?
thread | 与线程相关
thwait | 终止多个线程

### Unix

名称 | 作用
--- | ---
etc | 操作/etc/passwd等的库
syslog | UNIX syslog 接口

### MS Windows

名称 | 作用
--- | ---
win32ole | 跟 widnows 相关

### 正则表达式

名称 | 作用
---|---

### GC

名称 | 作用
---|---
profile | The GC profiler provides access to information on GC runs including time, length and object space size.
weakref | 生成可被GC回收的弱引用

### Design Pattern

名称 | 作用
--- | ---
delegate | 支持委托的类
forwardable | 向类中定义方法委托的功能
observer | Observer Pattern
singleton | Singleton Pattern

### 开发工具

名称 | 作用
--- | ---
benchmark | 衡量和报告用于执行 Ruby 代码的时间
debug | Ruby 调试器
logger | 提供了一个简单而复杂的日志记录工具，可以用它来输出消息
minitest | 单元测试
minitest/benchmark | 性能测试
minitest/spec | Rspec风格的测试
mkmf | 制作扩展库的工具
test/unit | unit 测试
tracer | Ruby 的 tracer 源代码级的跟踪

### 命令行

名称 | 作用
--- | ---
getoptlong | 命令行选项的解析
optparse | 命令行选项的解析

### 其他

名称 | 作用
--- | ---
dl | 动态链接库功能
e2mmap | 异常类和消息的映象
English | 给特殊变量 $! 等添加英文别名($ERROR_INFO 等)
extmk | ?
fiddle | ?
objspace | 扩展了ObjectSpace的模块，并增加了几个方法，以获取有关对象/内存管理的内部统计信息
ostruct | Python 式的 "attr on write" Struct
pp | Pretty-printer
prettyprint | PrettyPrint
racc | LALR（1）语法分析器生成器。It is written in Ruby itself, and generates Ruby programs.
racc/parser | Racc 运行时库
rake | ?
rdoc | This is the driver for generating RDoc output. It handles file parsing and generation of output.
ripper | ruby 脚本解析器
rubygems | ?
timeout | 处理超时的方法
tsort | 拓扑排序和强连接成分
uri | URI is a module providing classes to handle Uniform Resource Identifiers


## 参考

* [Ruby 2.1.5 Standard Library Documentation](http://www.ruby-doc.org/stdlib-2.1.5/)
* [Ruby Standard Library](https://github.com/rubysl)
* [Ruby标准库一览](http://my.oschina.net/u/130017/blog/15520)
