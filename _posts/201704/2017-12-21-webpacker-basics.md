---
layout: post
title:  "Webpacker 基础"
date:   2017-12-21 10:30:00
categories: rails
tags: rails5 javascript
author: "Victor"
---

## Webpack

学习之前先来了解一下前端面对的问题。

1. 全局作用域下容易造成变量冲突
2. 文件只能按照 `<script>` 的书写顺序进行加载
3. 开发人员必须主观解决模块和代码库的依赖关系
4. 大型项目中各种资源难以管理，长期积累的问题导致代码库混乱不堪

ECMAScript6 标准增加了 JavaScript 语言层面的模块体系定义。ES6 模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。这很棒棒，但是浏览器支持不太好。

```javascript
import "jquery";
export function doStuff() {}
module "localModule" {}
```

### 什么是 WebPack

WebPack 可以看做是模块打包机：它做的事情是，分析你的项目结构，找到 JavaScript 模块以及其它的一些浏览器不能直接运行的拓展语言（Sass，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。在3.0出现后，Webpack 还肩负起了优化项目的责任。

1. 打包：可以把多个 Javascript 文件打包成一个文件，减少服务器压力和下载带宽。
2. 转换：把拓展语言转换成为普通的 JavaScript，让浏览器顺利运行。
3. 优化：前端变的越来越复杂后，性能也会遇到问题，而 WebPack 也开始肩负起了优化和提升性能的责任。

`brew install webpack`

### Webpack 如何解决问题

前端开发过程中涉及到 JavaScript 模块、样式、图片、字体、HTML 模板等等众多的资源。以及这些资源的方言（LESS 等）。全都视作模块，通过 `require` 的方式来加载。

Webpack 编译的时候，对整个代码进行静态分析，分析出各个模块的类型和它们依赖关系，然后将不同类型的模块提交给适配的加载器来处理。比如一个用 `LESS` 写的样式模块，可以先用 `LESS` 加载器将它转成一个 `CSS` 模块，在通过 `CSS` 模块把他插入到页面的 `<style>` 标签中执行。

### 相关链接

* [Webpack 中文指南](http://zhaoda.net/webpack-handbook/index.html)
* [webpack 3 零基础入门教程](http://webpackbook.rails365.net/466996)
* [细说 webpack 之流程篇](http://taobaofed.org/blog/2016/09/09/webpack-flow/)
* [ECMAScript 6 入门](http://es6.ruanyifeng.com/)
* [官方 Loaders](https://webpack.js.org/loaders/)
* [官方 Plugins](https://webpack.js.org/plugins/)

## Webpacker

Wecpacker 是基于 Wecpack 3.x 的前端管理方案，你可以理解为它是代替原来 asset pipeline 的作用。

基本用法和特性就不讲了，直接看相关中的 webpack readme 就好。

需要注意的是，如果是用 `brew, yarn, npm` 全局安装的 webpack 是在 `/usr/local/bin/` 目录下，而用 rbenv 管理的 ruby 安装了 webpacker 之后，会在 `/Users/Victor/.rbenv/shims` 目录和 ruby 对应版本的 `bin` 目录下存在 webpacker 需版本的 webpack。

所以你可能需要在原来纯前端的项目下面执行 `/usr/local/bin/webpack entry.js output.js` 来编译你的文件。

### Yarn

默认使用 `yarn` 来管理 node modules，新添加一个 JS module 的方式：`yarn add bootstrap material-ui`

### Vue Props

Add the data as attributes in the element you are going to use (or any other element for that matter).

```
<%= content_tag :div,
  id: "hello-vue",
  data: {
    message: "Hello!",
    name: "David"
  }.to_json do %>
<% end %>
```

This should produce the following HTML:

```HTML
<div id="hello-vue" data="{&quot;message&quot;:&quot;Hello!&quot;,&quot;name&quot;:&quot;David&quot;}"></div>
```

Now, modify your Vue app to expect the properties.

```html
<template>
  <div id="app">
    <p>{{test}}{{message}}{{name}}</p>
  </div>
</template>

<script>
  export default {
    // A child component needs to explicitly declare
    // the props it expects to receive using the props option
    // See https://vuejs.org/v2/guide/components.html#Props
    props: ["message","name"],
    data: function () {
      return {
        test: 'This will display: '
      }
    }
  }
</script>

<style>
</style>
```

```javascript
document.addEventListener('DOMContentLoaded', () => {
  // Get the properties BEFORE the app is instantiated
  const node = document.getElementById('hello-vue')
  const props = JSON.parse(node.getAttribute('data'))

  // Render component with props
  new Vue({
    render: h => h(App, { props })
  }).$mount('#hello-vue');
})
```

## 相关

* [webpacker README](https://github.com/rails/webpacker)
