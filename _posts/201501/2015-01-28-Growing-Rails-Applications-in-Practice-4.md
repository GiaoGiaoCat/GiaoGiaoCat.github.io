---
layout: post
title:  "Growing Rails Applications in Practice - 4"
date:   2015-01-28 12:30:00
categories: rails
tags: learningnote refactoring
author: "Victor"
---

## Creating a system for growth - 下

### Taming stylesheets

你可能觉得在一个关于 Rails 的书里面读到 CSS 感到奇怪。

然而，我们发现可怕的 stylesheets 文件夹会让你的项目快速失控。

下面这些是不是听起来很熟悉？

* 你害怕修改样式，因为它会把你不可预知的某个显示效果破坏。
* 一个简单的 ``<label>`` 或者 ``<p>`` 就继承了无数的样式，你为了让他们符合你的要求要花大量时间来覆盖这些样式。
* 你经常不管三七二十一的使用 !important 好让自己的的样式表覆盖了其它的样式。

下面我们要讲一下是什么让 CSS 变得失去控制，我们要做些什么才能挽救它。

#### How CSS grows out of control

我们先通过一个简短的例子来说明 stylesheets 是如何屈服于阴暗面的。

我要要制作一个博客。首先我们给文章页面增加一点样式用来显示文章的题目和作者的头像。

我们可以用下面的 HTML 标记来描述这个布局：

```html
<div class="article">
  <img src="avatar.png" />
  <h1>Article title</h1>
  <p>Lorem ipsum...</p>
</div>
```

然后简单的增加点 CSS

```css
.article img {
  width: 200px;
  float: right;
}
```

通过浏览器看起来都不错，所以我们部署了第一个版本。

几分钟之后产品经理来个新需求：每个文章的最下面应该提供一个分享到 twitter 和 facebook 的按钮。

没问题，我们简单的增加一个 ``<div class='social'>`` 内容区用来包含这些链接就行。

```html
<div class="article">
  <img src="avatar.png" />
  <h1>Article title</h1>
  <p>Lorem ipsum...</p>
  <div class="social">
    <a href=".."><img src="twitter.png" /></a>
    <a href=".."><img src="facebook.png" /></a>
  </div>
</div>
```

因为 social 内的图片会继承上面的声明的属性，这让它们看起来很怪，因为居然是右对齐的，并且那么傻大。所以再增加点样式。

```css
.social img {
  width: auto;
  float: none;
}
```

现在看起来不错了，所以我们又部署了一个版本。

接下来新需求来了。首页需要有一个列表显示最近的文章。并且首页的列表不需要显示作者头像。因为它会分散用户对主页 banner 的注意力。没问题，我们就在 ``<body>`` 标签上增加一个 ``homepage`` 的样式：

```html
<html>
  <body class="homepage">
    ...
  </body>
</html>
```

然后通过样式表来隐藏首页的作者头像：

```css
body.homepage .article img {
  display: none;
}
```

一切搞定，可以部署。但是没多久一个 bug 就汇报过来了。首页的分享按钮的图片也没了。我们很快就用下面的样式修复一下：

```css
.article .social img {
  display: inline !important;
}
```

虽然我们修复了 bug，但是现在 CSS 产生了尴尬的乱七八糟的依赖：

1. There is still the original style ``.article img``
2. Sometimes the attributes of that style are overshadowed by ``.social img``
3. In some contexts the ``<img>`` element gets hidden by ``body.homepage .article img``
4. The element’s visibility is restored by ``.social img``, by using ``!important``.

造成这个结果的原因是 CSS 是级联的。它的效果不如面向对象的编程中的继承。在 CSS 中每一个选择器的每一个属性都可能通过任何数量的其他选择器的属性所覆盖。所有这些选择器的副作用合在一起最后决定网站最终的样式。当你 stylesheet 中的样式越多这个问题越严重。

**可维护的样式表是很难的。**

任何理智的程序员都不会让他的 Ruby 代码这样工作。作为程序员，我们力争用简单的 APIs 来标明输入输出。我们尽可能的避免副作用和紧耦合。所以，对于 stylesheets 我们也要这样。

幸运的是我们有一个好办法。

#### An API for your stylesheets

如果让我们形象的描述出如何才是一个好的 CSS 结构。那么我们的清单看起来会这样：

* 限制样式之间互相影响
* 鼓励重用
* 限制重复的代码
* 简单的查看有哪些样式可用
* 对去哪查看已经存在的样式有明确的规则
* 对如何添加新样式有明确的规则
* 可以重构样式而不用担心删除或破坏已有的效果

BEM(short for Block, Element, Modifier) 是一个有效的解决上面的问题系统方法。BEM 不是下载一个类库。它是一组关于样式表结构的规则。

BEM 可以为你那拥有 10000 行 PHP 代码的项目提供干净利落的领域模型。它划分责任，并为样式提供清晰的 API。最重要的是它有清晰地界限以描述可能发生的副作用。

首先，BEM 看起来很有效。你必须问自己为什么你不能随便乱写而需要约束自己的样式表。但是就像 spaghetti 对比 OOP 一样，一旦使用 BEM 那么它带来的好处很快变得清晰。在使用 BEM 一周后，如果按照过去的方式写 CSS 你会觉得自己很傻且不专业。

#### Blocks

在 BEM 中你的每个样式都是统一的。BEM 样式由一个简单，扁平的 **block** 列表组成，而不是疯狂的到处添加样式。

Here are some examples for blocks:

* A navigation bar
* A blog article
* A row of buttons
* Columns that divide the available horizontal space

你可以把 blocks 想象成样式表中的 ``classes`` 。Blocks 通过简单的 CSS 选择器来是实现：

```css
.navigation { ... }
.article { ... }
.buttons { ... }
```

#### Elements

组件通常比单一的样式规则更复杂。这就是为啥 BEM 提出了 **elements** 的概念：

Here are some examples for elements:

* The block ``navigation bar`` comprises multiple ``section`` elements
* The block ``blog article`` comprises an element ``title`` and an element ``text``
* The block ``columns`` contains multiple ``column`` elements

把 elements 想象成你样式表中 classes (blocks) 的方法。Elements 可以使简单的 CSS 选择器或者有前缀的 block 名：

```html
<div class="article">
  <div class="article__title">
    Awesome article
  </div>
  <div class="article__text">
    Lorem ipsum dolor ...
  </div>
</div>
```

下面是这些元素的样式表：

```css
.article { ... }
.article__title { ... }
.article__text { ... }
```

#### Modifiers

#### The BEM prime directive

#### Full BEM layout example

#### Organizing stylesheets

#### BEM anti-patterns

#### Living style guides

#### Pragmatic BEM

