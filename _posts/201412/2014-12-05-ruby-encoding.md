---
layout: post
title:  "Ruby 基础教程第4版读书笔记 7 - Encoding"
date:   2014-12-05 15:40:00
categories: ruby
tags: learningnote
author: "Victor"
---

## Ruby 的编码与字符串

* Ruby 的每个字符串对象都包含 *字符串数据本身* 以及 *该数据的字符编码* 两个信息。
* 数据的获取方式决定了它的编码方式。

### Ruby 按照如下信息决定字符串对象的编码，或者在输入输出时候转换编码。

* 脚本编码。决定字面量字符串对象编码的信息。
* 内部编码与外部编码。
  * 内部编码指从外部获取的数据在程序中如何处理的信息。
  * 外部编码指程序向外部输出时与编码相关的信息。

## 脚本编码与魔法注释

* Ruby 的脚本编码就是通过在脚本的开头书写魔法注释来指定的。
* 脚本自身的编码称为脚本编码。
* 脚本中的字符串、正则表达式的字面量会依据脚本编码进行解释。
* 魔法注释必须写在第一行，除非第一行是 **shebang**
* 程序内部写了与脚本编码不一致的代码会产生错误。
* 使用特殊字符 ```\u``` 创建字符串后，不管脚本编码的类型，都会生成 utf-8 的字符串，因此可以使用 ```encode!``` 进行转换。

```ruby
# encoding: EUC-JP
a = '\u3042\u3044'
puts a #=> "あい"
p a.encoding #=> <Encoding:UTF-8>
a.encode!("EUC-JP")
p a.encoding #=> <Encoding:EUC-JP>
```

## Encoding 类

* ```String#encoding``` 方法返回 **Encoding** 对象。
* ```String#encode``` 方法转换字符串对象的编码。在脚本中使用不同的编码时，需要进行必要的转换时可用次方法。
* 操作字符串时，Ruby 会自动进行检查。
* 为防止在连接不同编码的字符串会产生错误，必须用 ```encode``` 把两者转换为相同的编码。
* 在进行字符串比较时，如果编码不一样，即便表面的值相同，程序也判断为不相同的字符串。

```ruby
# encoding: utf-8
str = "あい"
p str.encoding #=> <Encoding:UTF-8>
str2 = str.encode("EUC-JP")
p str2.encoding #=> <Encoding:EUC-JP>
str + str2 #=> Encoding::CompatibilityError: incompatible character encodings: UTF-8 and EUC-JP
```

### Encoding 类的方法

* ```Encoding.compatible?(str1, str2)``` 检查两个字符串的兼容性(指两个字符串是否可以连接)。
* ```Encoding.default_external``` 返回默认的外部编码
* ```Encoding.default_internal``` 返回默认的内部编码。
* ```Encoding.find(name)``` 返回编码名 name 对应的 Encoding 对象。
* ```Encoding.list``` 返回 Ruby 支持的编码一览表。
* ```Encoding.name_list``` 返回表示编码名的字符串一览表。
* ```enc.name``` 返回 Encoding 对象 enc 的编码名。
* ```enc.names``` 返回包含 Encoding 对象的名称一览表的数组。

```ruby
# encoding: utf-8
p Encoding.compatible?("AB".encode("EUC-JP"), "あ".encode("UTF-8")) #=> <Encoding:UTF-8>
p Encoding.compatible?("あ".encode("EUC-JP"), "あ".encode("UTF-8")) #=> nil

p Encoding.name_list
p Encoding.find("shift_jis").name #=> "Shift_JIS"
```

## ASCII-8BIT 与字节串

* **ASCII-8BIT** 是一个特殊的编码，用于表示二进制数据以及字符串。我们也称之为 **BINARY**。把字符串对象用字节串形式保存的时候也用这个编码。
* 该编码用在 ```Arrar#pack``` 方法将二进制数据生成为字符串时，```Marsha1.dump``` 方法将对象序列化后的数据生成为字符串时。
* 在使用 open-uri 库从网络读取数据时，不知道字符编码是什么也用这个编码。

pack 方法的参数为字节串化时使用的模式，C4 表示 4 个 8 位的不带符号的整数。执行结果为 4 个字节的字节串，编码为 ASCII-8BIT。

```ruby
str = [127, 0, 0, 1].pack("C4")
p str #=> "\x7F\x00\x00\x01"
p str.encoding #=> <Encoding:ASCII-8BIT>
```

```ruby
# encoding: utf-8
require 'open-uri'
str = open("http://www.example.jp/").read
p str.encoding #=> <Encoding:ASCII-8BIT>
str.force_encoding("Windows-31J")
p str.encoding #=> <Encoding:Windows-31J>
```

* 即便编码为 **ASCII-8BIT** 的字符串也还是正常的字符串，只要知道字符编码，就可以使用 ```force_encoding``` 方法。这个方法并不会改变字符串的值(二进制的值)，而只改变编码信息。
* 使用 ```force_encoding``` 方法时，即使指定的编码不正确也不会立即报错，只有在对该字符串进行操作时候才产生错误。检查编码是否正确可以使用 ```valid_encoding?``` 方法。

```ruby
# encoding: UTF-8
str = "あい"
str.force_encoding("US-ASCII")
p str.valid_encoding? #=> false
str + "あ" #=> Encoding::CompatibilityError: incompatible character encodings: US-ASCII and ASCII-8BIT
```

## 正则表达式与编码

用不同编码进行正则匹配就会产生错误。通常情况下，正则表达式的字面量的编码与代码的编码是一样的。想指定其它编码可以使用 ```Regexp#new``` 方法。

```ruby
str = "模式".encode("EUC-JP")
re = Regexp.new(str)
p re.encoding #=> <Encoding:EUC-JP>
```

## IO 类与编码

* 外部编码指的是作为输入、输出对象的文件、控制台等的编码。
* 内部编码指的是脚本编码。

### 外部编码与内部编码

* 没有明确指定编码时，IO 对象的外部编码与内部编码各自使用其默认值 ```Encoding.default_external, Encoding.default_internal```。
* 默认情况下，外部编码基于各个系统的本地信息设定，内部编码不设定。

```ruby
p Encoding.default_external #=> Encoding:US-ASCII
p Encoding.default_internal #=> nil

File.open("foo.txt") do |f|
  p f.default_external #=> Encoding:US-ASCII
  p f.default_internal #=> nil
end
```

### 编码的设定

* 编码只用来说明如何处理字符的信息，因此对文本文件以外的文件没啥作用。
* ```IO#seek, IO#read(size)``` 方法都不受编码影响。
* ```IO#read(size)``` 方法读取的字符串的编码为表示二进制数据的 **ASCII-8BIT**。

可以通过如下两种办法设定 IO 的编码信息。

* ```io.set_encoding(encoding)```
* ```FIle.open(file, "mode:encoding")```

```ruby
$stdin.set_encoding("Shift_JIS:UTF-8")
p $stdin.default_external #=> <Encoding:Shift_JIS>
p $stdin.default_internal #=> <Encoding:UTF-8>
```

```ruby
# 指定外部编码
File.open("foo.txt", "w:UTF-8")

# 指定外部编码和内部编码
File.open("foo.txt", "w:UTF-8:Shift_JIS")
```

### 编码的作用

执行环境和数据的编码一致，就不用考虑编码转换问题。若不一致，就要在程序里面有意识的处理编码问题。

#### 输出时编码的作用

* 外部编码影响 IO 的写入（输出）。在输出的时候，会基于每个字符串的原编码和 IO 对象的外部编码进行编码的转换。因此输出用的 IO 对象不需要指定内部编码。
* 如果没有设置外部编码，或者字符串的编码与外部编码一致，则不进行编码转换。在需要转换的时候，如果输出的字符串编码不正确程序就会抛出异常。

#### 输入时编码的作用

* 影响 IO 的读取（输入）。
* 如果外部编码没有设置，则会使用 Encoding.default_external 的值作为外部编码。
* 设定了外部编码，没设定内部编码时，将读取的字符串的编码设置为 IO 对象的外部编码。并不会进行编码转换，而是将文件，控制台输入的数据原封不动的保存为 String 对象。
* 外部内部编码都设定的时候，则会执行由外部编码转换为内部编码的处理。输入和输出的情况一样，在编码转换的过程中出错就抛出异常。
