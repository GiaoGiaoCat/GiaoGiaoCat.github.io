---
layout: post
title:  "Ruby 错误信息"
date:   2014-12-12 15:00:00
categories: ruby
tags: debug
author: "Victor"
---

### 常见异常

异常名 | 常见原因 | 怎样抛出
---|---|---
RuntimeError | raise抛出的默认异常 | raise
NoMethodError | 对象找不到对应的方法 | a=Oject.new; a.jackmethod
NameError | 解释器碰到一个不能解析为变量或方法名的标识符 | a=jack
IOError | 读关闭的流，写只读的流，或类似的操作 | STDIN.puts("不能写入")
Errno::error | 与文件IO相关的一类错误 | File.open(-10)
TypeError | 方法接受到它不能处理的参数 | a = 3 + "abc"
ArgumentError | 传递参数的数目出错 | def o(x) end; o(1,2,3)

## 常见异常的错误信息和排查方法

### syntax error

```
foo.rb:2 syntax error, unexpected kEND, expecting ')'
```

程序中又语法错误。特别是括号、字符串忘记关闭时，解析器报告错误的位置可能会比实际出错的位置靠前。遇到语法错误，请确认以下几点：

* if, while, begin 等是否存在对应的 end
* 括号、字符串是否已关闭
* Here Document 是否已关闭
* 数组、哈希的元素间的分隔符是否有错或者漏写
* 运算符的使用方法是否有错

### NameError/NoMethodError

方法或变量不存在。本例是由于将 return 写成了 retrun，因此产生异常。

```
name.rb:2:in `foo': undefined local variable or method `method' for main:Object (NameError) for name.rb:12 in `<name>'
```

```
name.rb:2:in `foo': undefined method `inject' for []:Array (noMethodError) for name.rb:12 in `<name>'
```

方法名有错误时，程序抛出 ```noMethodError``` 异常。这时请确认以下几点：

* 方法名、变量名的拼写是否有错
* 变量是否已赋值给对象
* 是否有将当前的类认作其它类

### ArgumentError

```
arg.rb:1:in `foo': wrong number of arguments (1 of 0) (ArgumentError) for arg.rb:4:in `<main>'
```

方法参数有错误。本例中对不需要参数的方法传递了 1 个参数，因此出现这个错误。


### TypeError

```
type:rb:1:in `scan': wrong argument type nil (expected Regexp) (TypeError) from type.rb:1:in `<main>'
```

将方法意料之外的类的对象传递给了方法。例如不小心把 nil 赋值给变量了。

### LoadError

```
load.rb:1:in `require': cannot load such file -- foo (LoadError) from load.rb:1:in `<main>'
```

指定给 require 的库无法引用。也有可能是使用中的库间接引用其他库时遇到的错误。这时应注意以下几点：

* require 的参数是否正确
* 需要读取的库是否已安装
* $LOAD_PATH 中的目录中是否有文件

### BUG

```
segv.rb:4: [BUG] Segmentation fault
ruby 2.0.0p0 (2013-02-24 reversion 39474) [x86_64-linux]

-- Control frame information ------------------------------
c:0004 p:---- s:0011 e:000010 CFUNC :segv
c:0003 p:0009 s:0007 e:000006 METHOD segv.rb:4
c:0002 p:0026 s:0004 E:0021f8 EVAL segv.rb:7 [FINISH]
c:0001 p:0000 s:0002 E:002368 TOP [FINISH]

---- DEBUG 信息 ----
```

Ruby 本身或其他扩展类引起的错误。建议升级 Ruby 版本，如果升级还存在，就反馈给 Ruby 开发团队然后坐等更新。

## 异常处理

### rescue 捕获异常

```ruby
begin
  result = 20 / 0
  puts result
rescue ZeroDivisionError
  puts "Zero error"
rescue
  puts "unkonw error"
end
```

### raise 抛出异常

```ruby
def divide(x)
  raise ArgumentError if x == 0
end

begin
  devide(0)
rescue ArgumentError
  ptus "ArgumentError"
end

#=> ArgumentError
```

### 自建异常类

```ruby
class ThrowExceptionLove < Exception
  puts "some error"
end

begin
  raise ThrowExceptionLove, 'got a error'
rescue ThrowExceptionLove => e
  puts "Error #{e}"
end

#=>
some error
Error got a error
```


## 参考

* [错误处理和异常](http://www.cnblogs.com/cnblogsfans/archive/2009/02/11/1388667.html)
