---
layout: post
title:  "如何写好文档"
date:   2019-04-04 13:00:00

categories: other
tags: knowledge
author: "Victor"
---

公式是一種思維框架，是一種經驗導向的方法論，將你過去的經驗總結和抽象，得到高度概括的因素。

世上有两件难事，第一是把别人的钱放在自己的口袋里，第二种是把自己的思想放进别人的脑袋里。而写作作为传递思想的一种方式，条理清晰是对我们最基本的要求。

## 基础

### 为什么写文档

* 项目维护需要
* 项目合作需要
* 思路整理需要
* 技术积累需要

简单来说，就是 总结 -> 积累 -> 提高 -> 复用 的过程。

### 用什么工具写

* Markdown 或 reStructuredText 都行。reStructuredText 有一点难用，但是更强大。
* 纯文本版本控制，所以我推荐 PlantUML 而不是 https://processon.com

### 如何写

1. 总分总，使用 MECE 和 金字塔 原则。
2. 一图胜前言，简单的 UML 就行。
3. 看人下菜，给运维的讲操作及逻辑，给非技术看尽量多图，给交接同事重点是架构设计和代码原理
4. 少即是多。文档太长更新累，看着也累。
5. 理论结合实际，最好配上简单的例子
6. 用 markdown 生成 PDF

### 什么是好文档

先看两个例子 [Rust 英文版](https://doc.rust-lang.org/stable/book/first-edition/index.html), [Rust 中文版](https://kaisery.gitbooks.io/rust-book-chinese/content/content/README%20%E4%BB%8B%E7%BB%8D.html)。

Rust 编程语言的官方文档，Mozilla 基金会专门花钱聘请来的文档编辑大神 Steve Klabnik，Steve Klabnik 是唯一一个因为文档写的好，写进 Rust 核心八人组的，然后一直负责撰写和维护 Rust 的各种文档。

### 技术文档的组成

它们最终描述的是：关键类，函数，API等等，它们应该列出函数，字段，属性和方法等内容，并列出如何使用它们。

#### 实体关系图（必选）

实体关系图（E-R图）是对功能抽象程度最高的文档。通过实体关系图，可以尽快了解设计者的思路。实体关系图的重点是看实体抽象是否正确，新的抽象能否正确实现所有用例。

构图要素：

1. 矩形框：表示实体，在框中记入实体名。
2. 菱形框：表示联系，在框中记入联系名。
3. 椭圆形框：表示实体或联系的属性，将属性名记入框中。对于主属性名，则在其名称下划一下划线。
4. 连线：实体与属性之间；实体与联系之间；联系与属性之间用直线相连，并在直线上标注联系的类型。（对于一对一联系，要在两个实体连线方向各写1； 对于一对多联系，要在一的一方写1，多的一方写N；对于多对多关系，则要在两个实体连线方向各写N,M。)

![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/03.png)
![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/04.png)

#### 用户角色权限表（可选)
![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/05.png)

#### 业务流程图（必选）
通过业务流程图，可以在大方向上知道产品的整体逻辑，业务流程图拆解可以得到任务流程图，任务流程图拆解可以得到页面流程图。
![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/06.png)

这一步从产品经理的角度来说，以泳道图的方式提供业务流程图。而从研发的角度来说，应该辅以时序图以体现代码具体的实现逻辑和思路。

#### 状态设计（必选）

系统设计中很大一个工作就是规划系统状态(数据)的分布，通过状态分布可以大致了解实现能达到的性能、一致性和鲁棒性。这份文档包括

* 新的功能会新增哪些状态(包括持久化状态和非持久化状态)，会对已有状态造成什么影响。
* 状态的格式(数据库的DDL或者no-sql的json/KV)
* 状态的分布(集中式,分片,对分片要指明Sharding方法)
* 状态的一致性方案(对不同状态的一致性需求，实时/定时, 推/拉, 读写分离等)
* 状态的存取(状态通过什么方式存取和暴露给外界，直接访问，消息，API等)

一般 Web Server 无状态，系统扩展性多半取决于状态分布，所以需要专门的状态设计文档详细阐述。 状态设计关注的重点是 **设计方案能否满足性能和扩展性需求** ，另外对C端系统还要考虑 **是否有高可用性方案**（放松一致性，提供可用性)

#### 系统交互（可选）

新功能牵涉到系统交互时，需要提供系统交互文档。系统交互文档重点描述系统间的数据流，这份文档包括

* 新功能牵涉到系统内部哪些模块，模块内的交互方式(API/MESSAGE/直接访问/etc.)
* 和哪些外部系统发生交互,包括引入的新系统以及之前有交互的老系统，采用什么具体的交互方式
* 交互接口是否有限制(性能/吞吐量/稳定性/etc.)
* 外部系统哪些是强依赖，哪些是弱依赖
* 数据流图，描述完成特定功能的闭环中，数据在各个系统(模块)间如何流转，从一个模块到另外一个模块的过程中，数据的形式如何转换。

通过系统交互文档，可以从更高的层次了解整个系统的复杂度和依赖。这里的重点是**数据流转过程中是否暴露了过多细节或引入了不必要的依赖**，评审的重点是数据流图有没有可能简化，将系统间的依赖降到最低

#### 接口文档（必选）

接口文档是接口两端程序员的约定(Contract)，任何需要多人合作的边界上都需要提供接口文档。

#### 全局说明（可选）
一些通用的控件、状态等，不需要每次都说明，比如空数据、网络异常、加载失败、刷新状态等等，只需说明一次即可。

#### 需求、功能、交互说明（必选）
以下几个维度去说明，将会让你考虑的更加全面：

* 字段、字段说明、数据来源
* 前置条件、排序机制、刷新机制
* 状态流转（一个页面可能有多个状态，需要说明）
* 交互操作（正常操作、异常操作）

这部分比较灵活，可以使用 Markdown 结合 MindNote 的方式。

#### 非功能需求（可选）

1. 埋点需求。页面的打开率、按钮点击率等，如果需要记录，则需要做说明。
2. 性能需求。请求数据的响应时间要求、并发数要求等。
3. 兼容性需求。系统版本的支持、多终端的支持、浏览器的支持等。

#### 修改记录（必选）
![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/07.png)
![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/08.png)

#### 使用 Git 和 Dropbox 来管理文档

1. 根目录索引文件，TB Card 号码对应功能简介，开发者，更新日期
2. 以 TB Card 号码创建子目录放置相关文档
3. 文件格式 Markdown、MindNote、PDF、PlantUML

## 相关知识
### MECE 原则

MECE 法则是指我们在分析问题时，把整体层层分解为要素的过程中，要遵循 **相互独立，完全穷尽** 的基本法则，确保每一层的要素之间 **不重叠，不遗漏**。

是把一些事物分成互斥（ME）的类别，并且不遗漏其中任何一个（CE）的分类方法。

可以使用 MECE 原则的一个案例是，将人们分成年龄不同的几类（已知他们各自的精确出生年份）。不能使用 MECE 原则的一个案例是，将人们分成国籍的几类（因为有些人没有国籍或具有两种及以上的国籍）。

#### MECE 的常用分类法

1. 二分法，把信息分成A和非A两个部分。
  * 比如国内、国外，他人、自己，已婚、未婚，成年人、未成年人，左右，男女，收入和支出，专业和业余等等。
2. 过程法，按照事情发展的时间、流程、程序，对信息进行逐一的分类。
  * 日常生活当中制定的日程表，解决问题的6个步骤，达成目标的3个阶段，其实都属于过程分类。过程分类法特别适合用于在对项目进展和阶段的汇报上。
3. 要素法，把一个整体分成不同的构成部分，可以是从上到下、从外到内等。这是一种整体和局部的关系。
  * 公司的组织架构图就是这种分类方法。
  * 人的身体分为头、躯干、四肢，电脑分为显示器和主机。
4. 公式法，公式法就是可以按照公式设计的要素去分类
  * 利润 = 销售额 - 成本
5. 矩阵法，也就是 4 象限法
  * 你要使用 2 次二分法
  * 重要紧急，重要不紧急，不重要但紧急，不重要也不紧急

![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/MECE.png)

### 金字塔原理

金字塔原理就是要先表明中心思想，再说论点、论据，然后层层延伸，状如金字塔。

#### 使用步骤

1. 结论先行，中心思想在最开头
2. 以上统下，上层思想是下层的概括
3. 归纳分组，每组思想属于一个统一的范畴
4. 逻辑递进，每组思想有一定的逻辑顺序
5. 补充：纵向疑问问答、横向四大顺序（结构、时间、总分、演绎）

![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/01.png)
![](http://wjp2013.github.io/assets/images/pictures/2019-04-04-how-to-write-document/02.png)

### 写作规范

* [Microsoft Manual of Style, 4th Edition](https://www.microsoftpressstore.com/store/microsoft-manual-of-style-9780735648715)
* [Mailchimp Content Style Guide](https://styleguide.mailchimp.com/)
* [The Yahoo! Style Guide](https://www.amazon.com/dp/B003P8QDFU/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)
* [Docker documentation](https://docs.docker.com/opensource/)
* [中文技术文档的写作规范](https://github.com/ruanyf/document-style-guide)
* [豌豆荚文案风格指南](https://github.com/wjp2013/the_room_of_exercises/blob/master/guides/%E8%B1%8C%E8%B1%86%E8%8D%9A%E6%96%87%E6%A1%88%E9%A3%8E%E6%A0%BC%E6%8C%87%E5%8D%97.md)

## 相关链接

* [《科技文档写作实务 人民邮电出版》](https://book.douban.com/subject/25784432/)
* [《Developing Quality Technical Information》](https://book.douban.com/subject/2851645/)
* [金字塔原理，看这一篇就够了！](http://www.woshipm.com/pmd/306704.html)
* [金字塔原理的解读与运用](https://www.sohu.com/a/209206336_444159)
* [写有价值的技术文档](https://yq.aliyun.com/articles/54013)
* [E-R图](https://baike.baidu.com/item/E-R%E5%9B%BE)
* [如何写一份程序员爱看的需求文档？](http://www.woshipm.com/pmd/778068.html)
