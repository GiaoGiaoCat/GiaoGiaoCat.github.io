---
layout: post
title:  "Venture Into Vim"
date:   2014-11-24 10:10:00
categories: tool
tags: editor
author: "Victor"
---

## Introduction

### Basic Movement

```
h, j, k, l
:w welcome.txt
:q
```

### Faster Movement

```
w, b, W, B, $, ^, 0, gg, G, {, }, f(*), F(*), t(*), T(*), (*)gg, (*)G, :(*)
3w, 2fb, tf, 15G
```

### Basic Editing

```
x, X, u, d(*), c(*), (*)dd, (*)cc, (*)i(*), (*)a(*),
ct", ci", ca'
```

### Cut, Copy and Paste

```
p, P, y(*), yy
10p
```

## Digging Deeper

### Search

```
/, ?, n, /., :noh
/.[aeiou], ?\n, d/(, c?d, y/)
```

### Replace

* 在 visual mode 下，s 仅替换选中区域
* gv 选中上一个 visual mode 下的区块
* gci 的意思是 global, confirm, inside, 也就是可以替换单词中的匹配字符
* \zs 指明匹配由此开始 :help /\zs, 同样也有 \ze
* 替换的流程是，先 /匹配，然后 visual mode 选取区块，然后 :s//replacement 替换

```
:s, (*)g, shift + v, %, :help (*), gv
:%s/pattern/replacement/gci, :s/pattern/replacement/, v%, :help :s, /[^ ](
```

想要验证自己是否掌握替换的用法，可以尝试能把下面的代码块

```
funciont()
// do something
function() {
  alert("Hello")
}
``

替换成

```
funciont()
// do something
function () {
  alert ("Hello")
}
```

## Vim as an Extension of Vi

## Customization


## 相关链接

* [Vimer的程序世界](http://www.vimer.cn/category/vim)
* [Vim应用大收集](http://be-evil.org/post-157.html)
