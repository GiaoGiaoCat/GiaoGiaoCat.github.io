---
layout: post
title:  "Go 中的字符串操作"
date:   2019-05-02 08:00:00

categories: go
tags: tip
author: "Victor"
---

## 基础

* rune 是 int32 的别名代表一个 UTF-8 字符，byte 是 unit8 的别名代表一个 ASCII 码字符。
* 通常字符字面量不能折行，需要折行要用反引号 `some string` 来处理。

### UTF-8 和 Unicode 的区别

* Unicode 是字符集，ASCII 也是字符集，a 在 Unicode 和 ASCII 中的编码都是 97，而 你 在 Unicode 的编码为 20320
* UTF-8 是编码规则，将 Unicode 中的字符 ID 以某种方式进行变长编码，1-4 个字节不等
	* 0xxxxxx 表示 0-127 兼容 ASCII 字符集
	* 从 128 到 0x10ffff 表示其它字符
* 广义 Unicode 指一个标准，定义字符集和编码规则

详见这篇总结 [位运算](/ruby/bitwise-operation-1/)

## 应用

### 计算字符串长度

* ASCII 字符长度使用 `len()` 函数
* Unicode 字符长度使用 `utf8.RuneCountInString` 函数

`len()` 函数可以用来获取切片、字符串、channel 等的长度。其返回值为 int 类型，对于字符串来说表示的是 ASCII 字符个数或字节长度。

```go
s1 := "genji is a ninja"
s2 := "忍者"
fmt.Println(len(s1)) // 16
fmt.Println(len(s2)) // 6
```

这里的差异是因为 Go 中的字符串都以 UTF-8 格式保存，每个中文占 3 个字节。如果想按照中文字符个数来计算，需要用 UTF-8 包提供的 `RuneCountInString()` 函数，统计 Unicode 字符数量。

```go
fmt.Println(utf8.RuneCountInString("忍者")) // 2
fmt.Println(utf8.RuneCountInString("忍者2")) // 3
```

### 遍历字符串

* 遍历 ASCII 字符串使用 for 的数值循环进行遍历，直接取每个字符串的下标获取 ASCII 字符
* Unicode 字符串遍历使用 for range

```go
theme := "狙击 start"
for i := 0; i < len(theme); i++ {
	fmt.Printf("ascii: %c %d\n", theme[i], theme[i])
}
```

```go
theme := "狙击 start"
for _, s := range theme {
	fmt.Printf("ascii: %c %d\n", s, s)
}
```

### 获取字符串的某一段字符

* strings.Index 正向搜索子字符串
* strings.LastIndex 反向搜索子字符串
* 搜索的起始位置可以通过切片偏移制作

```go
tracer := "死神来了，死神byebye"
comma := strings.Index(tracer, "，")
pos := strings.Index(tracer[comma:], "死神")

fmt.Println(comma, pos, tracer[comma+pos:])
```

### 修改字符串

* 修改字符串时，可以将字符串转换为 []byte 进行修改。
* []byte 和 string 可以通过强制类型转换互换。

Go 语言中的字符串和其它高级语言一样，默认是不可变的 immutable。所以我们无法修改字符元素，只能重新构造一个字符串并复制给原来的字符串变量来实现伪修改。代码实际修改的是 []byte，其是一个切片。字符串不变的好处很多，比如线程安全，只读无须加锁，方便内存共享，不必写时复制 Copy on Write，字符串哈希值也只需要制作一份。

```go
angel := "Heros never die"

angleBytes := []byte(angel)

for i := 5; i <= 10; i++ {
	angleBytes[i] = " "
}

fmt.Println(string(angleBytes))
```

### 连接字符串

使用 `+` 对字符串进行连接操作，非常直观，但不高效。

字符串也是一种字节数组，故此我们用 `bytes.Buffer` 它可以缓存并往里面写入各种字节数组。将需要连接的字符串，通过 `WriteString()` 方法，写入 `stringBuilder` 中，再通过 `stringBuilder.String()` 方法将缓冲转换为字符串。

```go
hammer := "吃我一拳"

sickle := "死吧"

var stringBuilder bytes.Buffer

stringBuilder.WriteString(hammer)
stringBuilder.WriteString(sickle)

fmt.Println(stringBuilder.String())
```

### 格式化输出

就是使用 fmt.Sprintf 了，文档对各个参数的用法说的很详细了。下面简单列一些常用的：


| 动词 | 功能                                                         |
| ---- | ------------------------------------------------------------ |
| %v   | 值的默认格式输出                                             |
| %+v  | 在 %v 的基础上，对结构体字段名和值进行展开                   |
| %#v  | Go 语言语法格式的值                                          |
| %T   | Go 语言语法格式的类型和值                                    |
| %%   | 输出 % 本体                                                  |
| %b   | 表示为二进制                                                 |
| %c   | 值对应的unicode码值                                          |
| %d   | 表示为十进制                                                 |
