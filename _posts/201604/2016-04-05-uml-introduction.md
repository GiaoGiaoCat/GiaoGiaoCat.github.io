---
layout: post
title:  "UML 入门"
date:   2016-04-04 12:00:00
categories: other
tags: uml
author: "Victor"
---

## 简介
统一建模语言是一种图形化的语言。为面向对象开发的系统要素进行说明、可视化、和编制文档的一种标准语言。

### 为什么要建模
简单的说是为了能够更好地理解正在开发的系统。

1. 有助于按照需求对系统进行可视化的分析
2. 能够系统的结构或行为
3. 给出了知道构造系统的模板
4. 对做出的决策进行文档化

## UML 的七种视图
UML 的七种视图各有各自的特点，各自有各自的作用。只有正确的认识七种视图才能对 UML 的九种图进行更加详细、深入的学习。UML 的九种图是七种视图的具体表现形式。

![](/assets/images/pictures/2016-04-05-uml-introduction/01.png)

## UML 的九种图

### 用例图 use case diagrams
* 概念：描述用户需求，从用户的角度描述系统的功能，强调这个系统是什么而不是这个系统怎么工作
* 描述方式：椭圆表示某个用例；人形符号表示角色
* 目的：帮组开发团队以一种可视化的方式理解系统的功能需求

![](/assets/images/pictures/2016-04-05-uml-introduction/001.png)

### 静态图
#### 类图 class diagrams
* 概念：显示系统的静态结构，表示不同的实体是如何相关联的
* 描述方式：三个矩形
* 目的：表示一个逻辑类或实现类，逻辑类通常是用户的业务所涉及的事物；实现类是程序员处理的实体

![](/assets/images/pictures/2016-04-05-uml-introduction/002.png)
![](/assets/images/pictures/2016-04-05-uml-introduction/003.png)

#### 对象图 object diagrams
* 概念：类图的一个实例，描述系统在具体时间点上所包含的对象以及各个对象的关系

![](/assets/images/pictures/2016-04-05-uml-introduction/004.png)

### 交互图
用来描述对象之间的交互关系

#### 序列图（顺序图/时序图）
* 概念：描述对象之间的交互顺序，着重体现对象间消息传递的时间顺序
* 描述方式：横跨图的顶部，每个框表示每个类的实例或对象；类实例名称和类名称使用冒号分开
* 目的：显示流程中不同对象之间的调用关系，还可以显示不同对象的不同调用

![](/assets/images/pictures/2016-04-05-uml-introduction/005.png)

#### 协作图 Collaboration diagrams
* 概念：描述对象之间的合作关系，侧重对象之间的消息传递

### 行为图
描述系统的动态模型和对象之间的交互关系

#### 状态图 Statechart diagrams
* 概念：描述对象的所有状态以及事件发生而引起的状态之间的转移
* 描述方式：
  1. 起始点：实心圆
  2. 状态之间的转换：使用开箭头的线段
  3. 状态：圆角矩形
  4. 判断点：空心圆
  5. 一个或多个终止点：内部包含实心圆的圆
* 目的：表示某个类所处的不同状态以及该类在这些状态中的转换过程

#### 活动图 Activity diagrams
* 概念：描述满足用例要求所要进行的活动以及活动时间的约束关系
* 描述方式：
  1. 起始点：实心圆
  2. 活动：圆角矩形
  3. 终止点：内部包含实心圆的圆
  4. 泳道：实际执行活动的对象
* 目的：表示两个或多个对象之间在处理某个活动时的过程控制流程

![](/assets/images/pictures/2016-04-05-uml-introduction/006.png)

活动图和状态图区别：

![](/assets/images/pictures/2016-04-05-uml-introduction/007.png)

### 实现图
#### 组件图 Component diagrams
组件是代码模块。组件图是是类图的物理实现。

* 概念：描述代码构件的物理结构以及各构件之间的依赖关系
* 描述方式：构件
* 目的：提供系统的物理视图，根据系统的代码构件显示系统代码的整个物理结构

![](/assets/images/pictures/2016-04-05-uml-introduction/008.png)

#### 部署图(配置图) Deployment diagrams
配置图则是显示软件及硬件的配置。

* 概念：系统中硬件的物理体系结构
* 描述方式：
  1. 三维立方体表示部件
  2. 节点名称位于立方体上部
* 目的：显示系统的硬件和软件的物理结构

![](/assets/images/pictures/2016-04-05-uml-introduction/009.png)

物理上的硬件使用节点nodes表示。每个组件属于一个节点。组件用左上角带有两个小矩形的矩形表示。

下面的配置图说明了与房地产事务有关的软件及硬件组件的关系。

![](/assets/images/pictures/2016-04-05-uml-introduction/001.gif)

## 相关阅读

* [用例图、顺序图、状态图、类图、包图、协作图](http://blog.csdn.net/zfrong/article/details/4086424)
* [UML用例图总结](http://blog.csdn.net/tianhai110/article/details/6369762)
* [UML序列图总结](http://blog.csdn.net/tianhai110/article/details/6361338)
* [10个uml学习网站](http://www.nnbaike.com/Course/497.html)
* [10个uml教程](http://www.nnbaike.com/Course/498.html)
* [UML 教程](http://www.sparxsystems.cn/resources/tutorial/uml-tutorial.html)
* [UML概览](http://www.uml.org.cn/oobject/OObject.asp)
