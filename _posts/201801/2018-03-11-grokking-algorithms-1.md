---
layout: post
title:  "图解算法 1"
date:   2018-03-11 19:45:00

categories: other
tags: learningnote
author: "Victor"
---

## 基础知识
### 大O表示法

我们谈论算法速度时候，说的是随着输入的增加，其运行时间以什么样的速度增加。

大O表示法被用来描述一个算法的性能或复杂度。指出了在最糟糕情况下算法的运行时间。

从快到慢依次排列：

* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/7b8d37856ce892bd7ee9a9f58ffea0febee5e9bc) 常量时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/6c35c6c21c30a5643d3100b7993f907b58cf79cf) 对数时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/8bc936ac28af050e96d7262b89eb11af36bcc958) 线性时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/b4a9cde84a808a1c8b6658032611f99e7fa0bb13) 二次方时间
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/a830bd28003689e75eb85a330c2017413dcceb98) 指数时间，这里 c 是算法所需的固定时间，被称为常量
* ![](https://wikimedia.org/api/rest_v1/media/math/render/svg/f5a7957bb5d704181437f3fcf22b257ecbe699a7) 阶乘时间

[大O小抄](http://bigocheatsheet.com/) 提供了常用的算法时间复杂度，并以图表的形式呈现。可以看一下 [维基百科](https://zh.wikipedia.org/wiki/%E5%A4%A7O%E7%AC%A6%E5%8F%B7)。

## 数据结构相关

内存中的存储形式可以分为连续存储和离散存储两种。数据的物理存储结构就有连续存储和离散存储两种，它们对应了我们通常所说的数组和链表。

### 数组

* **数据是连续的；随机访问速度块。**
* 数组是将元素在内存中连续存放，由于每个元素占用内存相同，可以通过下标迅速访问数组中任何元素。
* 但是如果要在数组中 增加/删除 一个元素，需要移动大量元素，在内存中空出一个元素的空间，然后将要增加的元素放在其中。

### 链表

* **优势在元素的插入和删除；读取所有元素速度块**
* 每个元素都存储了下一个元素的地址，从而使一系列随机元素的内存地址串在一起。

数组和链表的区别：

* 数组静态分配内存，链表动态分配内存
* 数组在内存中连续，链表不连续
* 数组元素在栈区，链表元素在堆区
* 数组利用下标定位，时间复杂度为O(1)，链表定位元素时间复杂度O(n)
* 数组插入或删除元素的时间复杂度O(n)，链表的时间复杂度O(1)

### 栈

* 有两种操作：压入和弹出
* 遵循后进先出（LIFO）
* 大部分功能函数顺序执行，且是可嵌套的，函数调用的顺序符合 “后入先出”（LIFO）顺序，所以用栈来记录

调用栈 call stack

* 所有函数调用都进入调用栈
* 调用栈可能很长，这将占用大量的内存
* 调用另一个函数时，当前函数暂停并处于未完成状态。该函数的所有变量的值都还在内存中

## 相关阅读

* [数据结构与算法（1）：数组与链表](https://plushunter.github.io/2017/07/18/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%EF%BC%881%EF%BC%89%EF%BC%9A%E9%93%BE%E8%A1%A8/)
