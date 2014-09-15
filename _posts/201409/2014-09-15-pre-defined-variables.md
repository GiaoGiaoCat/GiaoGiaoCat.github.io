---
layout: post
title:  "Ruby Pre-defined variables"
date:   2014-09-15 14:30:00
categories: ruby
tags: tip
author: "Victor"
---

ruby comes with a set of predefined variables.

```
$: = default search path (array of paths)
```

其他Ruby特殊变量：

* ```$!``` 最近一次的错误信息
* ```$@``` 错误产生的位置
* ```$_``` gets最近读的字符串
* ```$.``` 解释器最近读的行数(line number)
* ```$&``` 最近一次与正则表达式匹配的字符串
* ```$~``` 作为子表达式组的最近一次匹配
* ```$n``` 最近匹配的第n个子表达式(和$~[n]一样)
* ```$=``` 是否区别大小写的标志
* ```$/``` 输入记录分隔符
* ```$\``` 输出记录分隔符
* ```$0``` Ruby脚本的文件名
* ```$*``` 命令行参数
* ```$$``` 解释器进程ID
* ```$?``` 最近一次执行的子进程退出状态

### 相关链接

* [Ruby Programming/Syntax/Variables and Constants](http://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Variables_and_Constants)
* [Ruby Predefined Variables](http://www.tutorialspoint.com/ruby/ruby_predefined_variables.htm)
* [Pre-defined variables](http://web.njit.edu/all_topics/Prog_Lang_Docs/html/ruby/variable.html)
