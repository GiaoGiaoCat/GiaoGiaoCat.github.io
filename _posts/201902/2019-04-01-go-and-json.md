---
layout: post
title:  "Go and JSON"
date:   2019-04-01 13:00:00

categories: go
tags: tip
author: "Victor"
---

## 简介

### 什么是 JSON

JavaScript Object Notation 是一种轻量级的数据交换格式，因为易读性、机器容易处理而变得流行。

JSON 语言定义的内容非常简洁，主要分为三种类型：对象 object、数组 array 和基本类型 value。基本类型 value 包括：string、number、bool 和 null。

### Go 中使用

Golang 的 `encoding/json` 库已经提供了很好的封装，可以让我们很方便地进行 JSON 数据的转换。

Go 语言中数据结构和 JSON 类型的对应关系如下表：

| GOLANG 类型 | JSON 类型 | 注意事项 |
| --- | --- | --- |
| bool | JSON booleans | |
| 浮点数、整数 | JSON numbers | |
| 字符串 | JSON strings | 字符串会转换成 UTF-8 进行输出，无法转换的会打印对应的 unicode 值。<br> 而且为了防止浏览器把 json 输出当做 html <br> “<”、”>” 以及 “&” 会被转义为 “\u003c”、”\u003e” 和 “\u0026”。 |
| array，slice | JSON arrays | []byte 会被转换为 base64 字符串，nil slice 会被转换为 JSON null |
| struct | JSON objects | 只有导出的字段（以大写字母开头）才会在输出中 |

**Go 语言中一些特殊的类型，比如 Channel、complex、function 是不能被解析成 JSON 的。**

### Encode 和 Decode

静态类型语言解析 JSON 最大的问题是 JSON 主体中可能存在任何类型，编译器不知道如何设置内存来存放这些东西。对此，通常有两种解决方案：

1. 你提前知道 JSON 的数据格式，并定义好对应的结构体，任何不符合期望的字段都忽略掉。
2. ?

#### JSON to Struct

```go
type App struct {
    Id string `json:"id"`
    Title string `json:"title"`
}

data := []byte(`
    {
        "id": "k34rAT4",
        "title": "My Awesome App"
    }
`)

var app App
err := json.Unmarshal(data, &app)
```

#### Struct to JSON

```go
data, err := json.Marshal(app)
```

另一个例子：

```go
type User struct {
    Name string
    IsAdmin bool
    Followers uint
}

user := User{
        Name:      "cizixs",
        IsAdmin:   true,
        Followers: 36,
    }
data, err := json.Marshal(user)

// 那么 data 是 []byte 类型的数组，里面包含了解析为 JSON 之后的数据
data == []byte(`{"Name":"cizixs","IsAdmin":true,"Followers":36}`)
```

## Struct Tags



## 相关链接

* [JSON 官网](http://json.org/json-zh.html)
* [Go and JSON](https://eager.io/blog/go-and-json/)
* [encoding/json](https://golang.org/pkg/encoding/json)
* [Go by Example: JSON](https://gobyexample.com/json)
* [理解 Go 中的 JSON](https://sanyuesha.com/2018/05/07/go-json/)
* [Golang 中使用 JSON 的小技巧](https://colobu.com/2017/06/21/json-tricks-in-Go/)
* [Go 语言 JSON 简介](https://cizixs.com/2016/12/19/golang-json-guide/)
