---
layout: post
title:  "快应用入门对 CSS 的有限支持"
date:   2020-04-12 21:10:00

categories: quickapp
tags: tip
author: "Victor"
---

快应用对 CSS 的支持很奇怪，以为只是 CSS 3 的部分特性不支持，然后发现一些 CSS 2 的属性也不支持，这里记一下。

首先看一下这里 [通用样式](https://doc.quickapp.cn/widgets/common-styles.html)

## 选择器支持

1. id
2. class
3. tag
4. 并列，`a, .b`
5. 后代，直接后代 `.page .body text`

### 外部引入

```html
<style>
  @import '../Common/page1.css'
</style>
```

## Flex 布局

快应用 **容器** 标签，仅为 `div, list-item, tabs`。

容器属性

* flex-direction 不支持 *-reverse
* flex-wrap
* flex-flow 不支持
* justify-content 不支持 space-around
* align-items 不支持 baseline
* align-content

项目属性

* order 不支持
* flex-grow
* flex-shrink
* flex-basis 和 HTML5 特性不一致
* flex 和 HTML5 特性不一致
* align-self

## 坑

为字体设置的属性，必须放在 text 标签上，否则无效。

## 相关

* [项目结构讲解](https://doc.quickapp.cn/tutorial/overview/project-structure.html)

