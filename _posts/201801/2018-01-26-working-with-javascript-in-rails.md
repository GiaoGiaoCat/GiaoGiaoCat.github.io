---
layout: post
title:  "在 Rails 中使用 JavaScript"
date:   2018-01-26 17:10:00

categories: rails
tags: advanced
author: "Victor"
---

希望阅读本文的你，已经对基于 Rails 的 Web 开发有一定经验的。这意味着你必须熟悉 HTML 和 JavaScript。然后我们才能谈如何正确的在 Rails 中使用 JavaScript。

写下本文的时候，Rails 5.2 都 beta 了。时下，Rails 的前端生态又有一些变化，你需要掌握如下知识：

* HTML & HTML5，JavaScript & ES6
* [Yarn](/javascript/yarn-basics/)
* [Webpack & Webpacker](/rails/webpacker-basics/)
* [Turbolinks](/rails/turbolinks/)
* [rails-ujs](https://github.com/rails/rails/tree/master/actionview/app/assets/javascripts)
* Airbnb JavaScript Style Guide & [Reasonable System for JavaScript Structure (rsjs)](http://ricostacruz.com/rsjs/)

## Yarn

Yarn 对你的代码来说是一个包管理器，通过它来添加三方的 JavaScript lib。

## Webpakcer

干掉 Sprockets，改用 Webpacker。Webpacker 是在 Rails 中使用 Webpack 的方案。

## Ruby on Rails unobtrusive scripting adapter (rails-ujs)

Rails 使用一种叫做 “非侵入式 JavaScript”（Unobtrusive JavaScript）的技术把 JavaScript 依附到 DOM 上。

Rails 内置的 helper 方法，可以在 HTML 代码中插入 `data` 属性的，`rails-ujs` 提供相关的 JavaScript 代码，带来如下功能：

* 添加确认对话框
* 为普通的超链接生成 `non-GET` 请求
* 使用异步 Ajax 的方式提交表单或链接
* 当表单提交的时候可以自动禁用提交按钮

### 安装

`yarn add rails-ujs`

```javascript
import Rails from 'rails-ujs';
Rails.start()
```

### 用法

* 远程元素 `data-remote="true"`
* 定制远程元素 `data-method`, `data-url`, `data-params`, `data-type`
* 确认 `data-confirm`
* 自动禁用 `disable-with`

这里特别要注意的是，定制远程元素这一部分。这个功能很容易被忽略掉。

页面中有些元素并不指向任何 URL，但是却想让它们触发 Ajax 调用。为元素设定 `data-url` 和 `data-remote` 属性将向指定的 URL 发送 Ajax 请求。还可以通过 `data-params` 属性指定额外的参数，还可以通过 `data-type` 属性明确定义 Ajax 的 dataType。

例如，可以利用这一点在复选框上触发操作：

```html
<input type="checkbox" data-remote="true" data-url="/update" data-params="id=10" data-method="put">
```

其它例子：

```erb
# Confirmations
<%= link_to "Dangerous zone", dangerous_zone_path, data: { confirm: 'Are you sure?' } %>

# Automatic disabling
<%= form_with(model: @article.new) do |f| %>
  <%= f.submit data: { "disable-with": "Saving..." } %>
<%= end %>
```


## 参考

* [在 Rails 中使用 JavaScript](https://ruby-china.github.io/rails-guides/working_with_javascript_in_rails.html)
* [Turbolinks 5](http://wjp2013.github.io/rails/turbolinks/)
