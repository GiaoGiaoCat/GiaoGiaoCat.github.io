---
layout: post
title:  "CSS 的几个属性 display, float, clear, overflow, visibility"
date:   2014-09-09 09:55:00
categories: development
tags: css
author: "Victor"
---

### display

* block: 把行属性标签显示成块属性标签，可以设置宽高。
* inline: 把块属性标签显示成行属性标签，这时块属性标签就不能设置宽高。
* none: 使所控制的标签不显示。


### float

浮动，照样受文档流的限制。行标签 float 之后就可以设置它的宽高。

* none: 对象不浮动。
* left: 左浮动。
* right: 右浮动。

### clear

清除浮动

* none: 允许两边都可以有浮动对象。
* both: 不允许有浮动对象。
* left: 不允许左边有浮动对象。
* right: 不允许右边有浮动对象。

### overflow

超出

* visible: 不剪切内容也不添加滚动条。
* auto: 默认属性。
* hidden: 隐藏超出内容。
* scroll: 总是显示滚动条。

### visibility

可视

* inherit: 继承上一个父对象的可见性。
* visible: 对象可视。
* hidden: 对象隐藏。

#### visibility: hidden 和 display: none 的区别

1. hidden 是设置元素的框的不可见，但是在布局中的位置是不变的。会占用那个布局，只是不显示内容。会把不显示的元素给下载下来。
2. none 不会占用那个位置，下一个元素会直接覆盖它。不会把不显示的元素给下载下来。


### 几种图片格式的差别

* gif：不支持半透明。
* jpg：支持透明。
* png：部分支持透明，需要额外处理。


### 原文链接

* [CSS的几个属性display,float,clear,overflow,visibility](http://colobu.com/2014/08/28/CSS-display-float-clear-overflow-visibility/)
