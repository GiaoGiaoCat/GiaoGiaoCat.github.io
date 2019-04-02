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

JSON（JavaScript Object Notation） 是一种轻量级的数据交换格式，因为易读性、机器容易处理而变得流行。

JSON 语言定义的内容非常简洁，主要分为三种类型：对象（object）、数组（array）和基本类型（value）。基本类型（value）包括：string、number、bool 和 null。

### Go 中使用

Golang 的 `encoding/json` 库已经提供了很好的封装，可以让我们很方便地进行 JSON 数据的转换。

Go 语言中数据结构和 JSON 类型的对应关系如下表：

| GOLANG 类型	| JSON 类型	| 注意事项 |
| --- | --- | --- |
| bool | JSON booleans | |
| 浮点数、整数 | JSON numbers | |
| 字符串 | JSON strings | 字符串会转换成 UTF-8 进行输出，无法转换的会打印对应的 unicode 值。<br> 而且为了防止浏览器把 json 输出当做 html <br> “<”、”>” 以及 “&” 会被转义为 “\u003c”、”\u003e” 和 “\u0026”。 |
| array，slice | JSON arrays | []byte 会被转换为 base64 字符串，nil slice 会被转换为 JSON null |
| struct | JSON objects | 只有导出的字段（以大写字母开头）才会在输出中 |

### Encode 和 Decode

## 相关链接

* [JSON 官网](http://json.org/json-zh.html)
* [Go and JSON](https://eager.io/blog/go-and-json/)
* [encoding/json](https://golang.org/pkg/encoding/json)
* [Go by Example: JSON](https://gobyexample.com/json)
* [理解 Go 中的 JSON](https://sanyuesha.com/2018/05/07/go-json/)
* [Golang 中使用 JSON 的小技巧](https://colobu.com/2017/06/21/json-tricks-in-Go/)
* [Go 语言 JSON 简介](https://cizixs.com/2016/12/19/golang-json-guide/)
