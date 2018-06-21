---
layout: post
title:  "Ruby 基础教程第4版读书笔记 6 - File, Dir"
date:   2014-12-04 14:30:00
categories: ruby
tags: learningnote
author: "Victor"
---

## File 类

### 变更文件名

* ```File.rename(before, after)``` 变更文件名，如果文件移动到不存在目录，则会产生错误
* 无法跨文件系统或者驱动器移动文件
* 如果文件不存在或者没有相应权限则操作失败，程序抛出异常

```ruby
File.rename("before.txt", "backup/data.txt")
```

### 复制文件

Ruby 预定义的方法时无法复制文件的。只能只有方法组合来实现文件复制。

```ruby
def copy(from, to)
  File.open(from) do |input|
    File.open(to, "w") do |output|
      output.write(input.read)
    end
  end
end
```

也可以直接使用 FileUtils.cp 或者 FileUtils.mv

```ruby
require 'fileutils'
FileUtils.cp('data.txt', 'backup/data.txt')
FileUtils.mv('data.txt', 'backup/data.txt')
```

### 删除文件

* ```File.delete(file)```
* ```File.unlink(file)```

## Dir 类

### 获取当前目录和更改目录

* 程序可以获取运行时所在的目录信息。使用 Dir.pwd 方法获取当前目录，使用 Dir.chdir 方法变更当前目录。
* 当前目录下的文件，我们可以通过制定文件名直接打开，如果变更了目录，则还需要指定目录名。

```ruby
p Dir.pwd #=> "/usr/local/lib"
Dir.chdir("ruby/2.0.0") # 根据相对路径移动
p Dir.pwd #=> "/usr/local/lib/ruby/2.0.0"
Dir.chdir("/etc") # 根据绝对路径移动
p Dir.pwd #=> "/etc"
```

```ruby
p Dir.pwd #=> "/usr/local/lib"
io = File.open("ruby/2.0.0/find.rb")
io.close
```

### 目录内容的读取

* ```Dir.open(path)``` 打开目录
* ```Dir.close``` 关闭
* ```dir.read``` 程序遍历读取最先打开的目录下的内容。注意这里要忽略到当前目录和上级目录，否则会陷入死循环。
* ```Dir.glob``` 可以想 shell 一样使用 * 或者 ? 等通配符来取得文件名。并以数组形式返回。
* ```Dir.glob("*")``` 获取当前目录中的所有文件名。无法获取以 . 开头的隐藏文件。
* ```Dir.glob(".*")``` 获取当前目录中所有的隐藏文件名。
* ```Dir.glob(["*.html", "*.htm"])``` 获取当前目录中扩展名为 .html 和 .htm 的文件名，可通过数组指定多种模式。
* ```Dir.glob(%w(*.html *.htm))``` 同上
* ```Dir.glob(["*/*.html", "*/*.htm"])``` 获取子目录下扩展名为 .html 和 .htm 的文件名。
* ```Dir.glob("foo.[cho]")``` 获取文件名为 foo.c foo.h foo.o 的文件。
* ```Dir.glob("**/*")``` 获取当前目录及其子目录中所有的文件名，递归查找目录。
* ```Dir.glob("foo/**/*.html")``` 获取目录 foo 及其子目录中所有扩展名为 .html 的文件名，递归查找目录。

```ruby
dir = Dir.open("/usr/bin")
while name = dir.read
p name
end
dir.close
```

```ruby
dir = Dir.open("/usr/bin")
dir.each do |name|
p name
end
dir.close
```

```ruby
Dir.open("/usr/bin") do |dir|
dir.each do |name|
p name
end
end
```

```ruby
def traverse(path)
if File.directory?(path)
dir = Dir.open(path)
while name = dir.read
next if name == "."
next if name == ".."
traverse(path + "/" + name)
end
dir.close
else
process_file(path) # 处理文件
end
end

def process_file(path)
puts path
end

traverse(ARGV[0])
```

```ruby
def traverse(path)
Dir.glob(["#{path}/**/*", "#{path}/**/."]).each do |name|
unless File.directory?(name)
process_file(name)
end
end
end
```

### 文件与目录属性

* ```File.stat(path)``` 获取文件，目录的属性。返回的是 ```File::Stat``` 类的实例。
* ```File.utime(atime, mtime, path)``` 改变文件属性中的最后访问时间和最后修改时间。
* ```File.chmod(mode, path)``` 修改文件的访问权限，mode 的值为整数值。可以指定多个路径。
* ```File.chown(owner, group, path)``` 改变文件的所有者。owner 表示新的所有者的用户ID，group 表示新的所属组ID。执行这个命令需要有管理员的权限。

```ruby
filename = "foo"
File.open(filename, "w").close

st = File.stat(filename)
p st.ctime #=> 2013-03-03 04:20:01 +0900
p st.mtime #=> 2013-03-03 04:20:01 +0900
p st.atime #=> 2013-03-03 04:20:01 +0900

File.utime(Time.now - 100, Time.now - 100, filename)
st = File.stat(filename)
p st.ctime #=> 2013-03-03 04:20:01 +0900
p st.mtime #=> 2013-03-03 04:18:21 +0900
p st.atime #=> 2013-03-03 04:18:21 +0900
```

### FileTest 模块

模块中的方法用于检查文件的属性，可以 include 后使用，也可以作为模块函数使用。其中的方法也可以作为 File 类的类方法使用。

方法 | 返回值
---|---
exist?(path) | path 若存在则返回 true
file?(path) | path 若是文件则返回 true
directory?(path) | path 若是目录则返回 true
owned?(path) | path 的所有者与执行用户一样则返回 true
grpowned?(path) | path 的所属组与执行用户的所属组一样则返回 true
readable?(path) | path 可读取则返回 true
writable?(path) | path 可写则返回 true
executable?(path) | path 可执行则返回 true
size(path) | 返回 path 的大小
size?(path) | path 的大小比 0 大则返回 true，为 0 或者不存在则返回 nil
exist?(path) | path 若存在则返回 true
exist?(path) | path 若存在则返回 true

### 文件名的操作

* ```File.basename(path[, suffix])``` 返回路径 path 最后一个 "/" 以后的部分。如果制定扩展名，则会取出返回值中的扩展名部分。
* ```File.dirname(path)``` 返回路径 path 中最后一个 "/" 之前的部分。路径不包含 "/" 时则返回 "."。
* ```File.extname(path)``` 返回路径 path 中 basename 方法返回结果中的扩展名。没有扩展名或者以 "." 开头的文件名时则返回空字符串。
* ```File.split(path)``` 将路径分割为目录名与文件名两部分，并以数组形式返回。通常使用并行赋值的时候会使用这个方法。
* ```File.join(name1[, name2, ...])``` 用 File::SEPARATOR 连接参数制定的字符串。默认参数时 "/"
* ```File.expand_path(path[, default_dir])``` 根据目录名 default_dir 将相对路径转换为绝对路径。不指定 default_dir 时，则根据当前目录转换。

```ruby
p File.basename("/usr/local/bin/ruby") #=> "ruby"
p File.basename("src/ruby/file.c", ".c") #=> file
p File.basename("file.c") #=> file

p File.dirname("/usr/local/bin/ruby") #=> "/usr/local/bin"
p File.dirname("ruby") #=> "."
p File.dirname("/") #=> "/"

p File.extname("helloruby.rb") #=> ".rb"
p File.extname("ruby-2.0.0-p0.tar.gz") #=> ".gz"
p File.extname("img/foo.png") #=> ".png"
p File.extname("/usr/local/bin/ruby") #=> ""
p File.extname("~/.zshrc") #=> ""
p File.extname("/etc/init.d/ssh") #=> ""

p File.split("/usr/local/bin/ruby") #=> ["/usr/local/bin", "ruby"]
p File.split("ruby") #=> [".", "ruby"]
p File.split("/") #=> ["/", ""]
dir, base = File.split("/usr/local/bin/ruby")

p Dir.pwd #=> "/usr/local"
p File.expand_path("bin") #=> "/usr/local/bin"
P File.expand_path("../bin") #=> "/usr/bin"
P File.expand_path("bin", "/usr") #=> "/usr/bin"
P File.expand_path("../etc", "/usr") #=> "/etc"
p File.expand_path("~gotoyuzo/bin") #=> "/home/gotoyuzo/bin"
```

### Find 库

find 库中的 find 模块被用于对指定的目录下的目录或文件做递归处理。

* ```Find.find(dir) { |path| ... }``` 将目录 dir 下的所有文件路径逐个传给路径 path
* ```Find.prune``` 使用 find 方法时，调用 prune 方法后，程序会跳过当前目录下的所有路径
* ```Find.next``` 使用 find 方法时，调用 prune 方法后，程序会跳过当前目录，但子目录的内容继续查找

```ruby
require 'find'

IGNORES = [/^\./, /^CVS$/, /^RCS$/]

def listdir(top)
  Find.find(top) do |path|
    if FileTest.directory?(path)
      dir, base = File.split(path)
      IGNORES.each do |re|
        if re =~ base # 需要忽略的目录
          Find.prune # 忽略该目录下的内容的查找
        end
      end
      puts path # 输出结果
    end
  end
end

listdir(ARGV[0])
```

### tempfile 库

在处理大量数据时，程序会把一部分正在处理的数据写入到临时文件。这些文件在程序执行完之后就不在需要，因此必须删除。而临时文件的名字也要不相同。 tempfile 就是用来解决这个问题。

* ```Tempfile.new(basename[, tempdir])``` 创建临时文件，文件名格式为 "basename + 进程id + 流水号"。如果不指定目录名，则会按照顺序查找 ```ENV["TMPDIR"], ENV["TMP"], ENV["TEMP"], /tmp```
* ```tempfile.close(real)```
* ```tempfile.open```
* ```tempfile.path```

### fileutils 库

* ```FileUtils.cp(from, to)``` 把文件从 from 复制到 to，to 为目录时，则在 to 下面建立与 from 同名的文件。
* ```FileUtils.cp_r(from, to)```
* ```FileUtils.mv(from, to)```
* ```FileUtils.rm(from, to)``` 删除 path，path 只能为文件，可以一次删除多个文件。执行处理中遇到错误就中断处理。
* ```FileUtils.rm_f(from, to)``` 同上，但是会忽略错误继续执行。
* ```FileUtils.rm_r(from, to)```
* ```FileUtils.rm_rf(from, to)```
* ```FileUtils.compare(from, to)```
* ```FileUtils.install(from, to)```
* ```FileUtils.mkdir_p(from, to)```
