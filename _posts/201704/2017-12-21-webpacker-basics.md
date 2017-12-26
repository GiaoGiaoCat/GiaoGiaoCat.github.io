---
layout: post
title:  "Webpacker 基础"
date:   2017-12-21 10:30:00
categories: rails
tags: rails5 javascript
author: "Victor"
---

Wecpacker 是基于 Wecpack 3.x 的前端管理方案，你可以理解为它是代替原来 asset pipeline 的作用。

基本用法和特性就不讲了，直接看相关中的 webpack readme 就好。

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
