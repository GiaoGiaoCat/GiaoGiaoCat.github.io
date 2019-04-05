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

* 你将会在 6 个月后使用你的代码
* 你想要别人使用你的代码
* 你需要别人的帮助
* 能提升代码的设计，整理自己的思路
* 让别人按照你原来的意图贡献代码

### 用什么工具写

* Markdown 或 reStructuredText 都行。reStructuredText 有一点难用，但是更强大。
* 纯文本版本控制，所以我推荐 PlantUML 而不是 https://processon.com

### 如何写

## 什么是好文档

先看两个例子 [Rust 英文版](https://doc.rust-lang.org/stable/book/first-edition/index.html), [Rust 中文版](https://kaisery.gitbooks.io/rust-book-chinese/content/content/README%20%E4%BB%8B%E7%BB%8D.html)。

Rust 编程语言的官方文档，Mozilla 基金会专门花钱聘请来的文档编辑大神 Steve Klabnik，Steve Klabnik 是唯一一个因为文档写的好，写进 Rust 核心八人组的，然后一直负责撰写和维护 Rust 的各种文档。


### 一些要点

* 有章节，修订记录，目标关注，页眉页脚，附录
* 生成 PDF



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

### 中文写作规范

## 相关链接

* [《科技文档写作实务 人民邮电出版》](https://book.douban.com/subject/25784432/)
* [《Developing Quality Technical Information》](https://book.douban.com/subject/2851645/)
* [推荐 | 金字塔原理，看这一篇就够了！](http://www.woshipm.com/pmd/306704.html)
* [金字塔原理的解读与运用](https://www.sohu.com/a/209206336_444159)
