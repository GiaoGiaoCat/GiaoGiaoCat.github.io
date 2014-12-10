---
layout: post
title:  "开发环境配置补遗"
date:   2014-12-10 11:10:00
categories: tool
tags: editor
author: "Victor"
---

## Dash

1. 安装各种需要的文档。
2. 修改 Docset keywords，比如 Ruby2 默认的 keyword 是 "ruby:"，我会修改成 "r2:"
3. 创建自己的 Search Profiles，比如当我打开 ST3 的时候我肯定是在写 Ruby 代码，这时候我需要搜索 Ruby 和 Rails 的文档，那么就创建一个这样的 Profile，当 ST3 激活的时候，默认仅搜索这两个文档。
4. 在编辑 Search Profiles 的时候，给默认的 Profile 添加一个 Search Keyword，我取名叫 'all:'。
5. 在编辑器中安装 Dash 的支持，例如 ST3 的 DashDoc，这样我可以使用快捷键 ```Ctrl + Option + h``` 在 Dash 中搜索当前选中单词。
6. Dash 的快捷键
  * ```ALT+Down/Up``` 在当前页面的左下方的方法列表中移动。
  * ```CMD+F``` 在当前页面中搜索下一个符合条件的结果。
  * 搜索框中输入 ```String.new``` 意思要搜索 ```String``` 类或模块内的 ```new``` 方法。
  * 搜索框中输入 ```String new``` 意思要搜索 ```String``` 类或模块内的 ```new``` 关键词。

* [Dash 帮助手册](http://kapeli.com/guide/guide.html()
