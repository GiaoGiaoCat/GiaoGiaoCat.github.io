---
layout: post
title:  "常用的 Google 搜索技巧"
date:   2017-11-29 11:30:00
categories: tool
tags: tip
author: "Victor"
---

## 关键词用法

### 基本搜索 `+, -, OR`

| 运算符 | 用途 | 实际用法实例 |
| --- | --- | --- |
| + | 包含关键词 keyword1 和 keyword2 | `keyword1 keyword2` 或者 `keyword1+keyword2` |
| - | 包含关键词 keyword1 而不含 keyword2 | `keyword1 -keyword2` |
| OR | 包含关键词 keyword1 或者 keyword2 | `keyword1 OR keyword2` |

### 高级搜索 `site, filetype, link, inurl, allinurl, intitle, allintitle`

| 运算符 | 用途 | 实际用法实例 |
| --- | --- | --- |
| site | 局限于某个具体网站 | `金庸 site:edu.cn` |
| filetype | 查询某一类文件 | `郭靖 filetype:pdf` |
| link | 返回所有链接到某个 URL 地址的网页 | `link:www.newhua.com` |
| inurl | 返回的网页链接中包含第一个关键字，后面的关键字则出现在链接中或者网页文档中 | `inurl:java jvm site:ibm.com` |
| allinurl | 返回的网页的链接中包含所有作用关键字 | `allinurl:java jvm` |
| intitle | 同上，但出现在网页标题中 | `intitle:郭靖` |
| allintitle | 同上，但出现在网页标题中 | `allintitle:郭靖` |

### 其它少用的关键词 `link, related, cache`

| 运算符 | 用途 | 实际用法实例 |
| --- | --- | --- |
| link | 搜索所有链接到某个 URL 地址的网页 | `link:www.521java.com` |
| related | 搜索结构内容方面相似的网页 | `related:www.baidu.com` |
| cache | 搜索 Google 服务器上某页面的缓存 | `cache:www.521java.com` |

### 注意

* Google 不支持 `*` 通配符，对大小写也不敏感。
* 对一些网路上出现频率极高的英文单词，如 i、com、www 等，以及一些符号如 `*`、`.` 等，会作忽略处理。除非前面加上明文的 `+` 符号。

## 相关链接

* [Google高级搜索技巧](http://blog.csdn.net/xlinsist/article/details/47380049)
* [程序员必须要学会Google搜索技巧](http://blog.csdn.net/u013501637/article/details/52658726)
* [如何用好谷歌等搜索引擎？](https://www.zhihu.com/question/20161362)
