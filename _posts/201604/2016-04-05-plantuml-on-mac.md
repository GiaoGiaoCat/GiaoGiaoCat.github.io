---
layout: post
title:  "在 Mac 上使用 PlantUML 画 UML"
date:   2016-04-04 20:00:00
categories: tool
tags: unl
author: "Victor"
---

## 什么是 PlantUML

PlantUML 是一个画图脚本语言，用它可以快速地画出各种 UML 图。

## 安装
我最近一年主力编辑器是 Atom，所以下面的演示方案也是基于 Atom

```
brew install graphviz
apm install language-plantuml
apm install plantuml-viewer
```

## 简单使用
使用的话比较简单，绘图的内容需要包含在 `@startuml` 和 `@enduml` 中，不然会报错。

```
@startuml
Bob -> Alice : Hello, how are you
Alice -> Bob : Fine, thank you, and you?
@enduml
```

## 相关链接

* [使用 Sublime + PlantUML 高效地画图](http://www.jianshu.com/p/e92a52770832)
* [官方中文文档](http://translate.plantuml.com/zh)  
