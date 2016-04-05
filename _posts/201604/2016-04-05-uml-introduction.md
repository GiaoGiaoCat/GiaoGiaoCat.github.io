---
layout: post
title:  "UML 入门"
date:   2016-04-04 12:00:00
categories: other
tags: uml
author: "Victor"
---

## Unified Modeling Language
统一建模语言是一种图形化的语言。为面向对象开发的系统要素进行说明、可视化、和编制文档的一种标准语言。

### 为什么要建模
简单的说是为了能够更好地理解正在开发的系统。

1. 有助于按照需求对系统进行可视化的分析
2. 能够系统的结构或行为
3. 给出了知道构造系统的模板
4. 对做出的决策进行文档化

## UML 的七种视图
UML 的七种视图各有各自的特点，各自有各自的作用。只有正确的认识七种视图才能对 UML 的九种图进行更加详细、深入的学习。UML 的九种图是七种视图的具体表现形式。

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-uml-introduction/01.png)

## UML 的九种图

### 用例图 use case diagrams
* 概念：描述用户需求，从用户的角度描述系统的功能
* 描述方式：椭圆表示某个用例；人形符号表示角色
* 目的：帮组开发团队以一种可视化的方式理解系统的功能需求

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-uml-introduction/001.png)

### 静态图
#### 类图 class diagrams
* 概念：显示系统的静态结构，表示不同的实体是如何相关联的
* 描述方式：三个矩形
* 目的：表示一个逻辑类或实现类，逻辑类通常是用户的业务所涉及的事物；实现类是程序员处理的实体

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-uml-introduction/002.png)
![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-uml-introduction/003.png)

#### 对象图 object diagrams
* 概念：类图的一个实例，描述系统在具体时间点上所包含的对象以及各个对象的关系

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-uml-introduction/004.png)

### 交互图
用来描述对象之间的交互关系

#### 序列图（顺序图/时序图）
* 概念：描述对象之间的交互顺序，着重体现对象间消息传递的时间顺序
* 描述方式：横跨图的顶部，每个框表示每个类的实例或对象；类实例名称和类名称使用冒号分开
* 目的：显示流程中不同对象之间的调用关系，还可以显示不同对象的不同调用

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-uml-introduction/005.png)

#### 协作图 Collaboration diagrams
* 概念：描述对象之间的合作关系，侧重对象之间的消息传递
