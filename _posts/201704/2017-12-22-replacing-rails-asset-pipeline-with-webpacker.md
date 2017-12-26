---
layout: post
title:  "使用 Webpacker 替换 Sprockets"
date:   2017-12-21 11:30:00
categories: rails
tags: rails5 javascript
author: "Victor"
---

## 前言

以前为了找一个 js 或者 css 包时，只能想尽办法找到这个包的 gem 或者直接弄到 vendor 目录下。5.1 版加入了 yarn 之后的做法就简单多了：

1. `yarn add jquery`
2. 在 `assets/javascript.js` 中，加入 `#= require jquery/dist/jquery`


## 从 0 开始

先创建一个使用 Webpacker 替换 Sprockets 的 Rails 项目 `rails new blank --skip-sprockets --webpack`。

你会发现 Gemfile 中 `sass-rails, uglifier & coffee-rails` 都没了。`app/assets` 目录仍然存在，但如果你开发的是一个单页型应用的话，这个目录你也可以考虑删除，因为大部分工作都在 `app/javascript` 目录下。

下面看看 Webpacker 默认生成的 `application.js`。

```javascript
/* eslint no-console:0 */
// This file is automatically compiled by Webpack, along with any other files
// present in this directory. You're encouraged to place your actual application logic in
// a relevant structure within app/javascript and only use these pack files to reference
// that code so it'll be compiled.
//
// To reference this file, add <%= javascript_pack_tag 'application' %> to the appropriate
// layout file, like app/views/layouts/application.html.erb

console.log('Hello World from Webpacker')
```

### Directory structure

我的建议是以功能或模块的方式在 `app/javascript` 下建立子文件夹，下面是一个例子：

```
my_project
+-- app
|   +-- javascript
|   |   +-- blog
|   |   |   +-- fonts
|   |   |   +-- images
|   |   |   +-- styles
|   |   |   +-- index.js
|   |   +-- packs
|   |   |   +-- application.js
```

再看看 `app/javascript/packs/application.js` 的入口怎么写：

```javascript
import 'blog';
```

这句的意思是引入并执行 `app/javascript/blog/index.js`，该文件是 JavaScript 应用的入口。

最后在 layout 中引入 `javascript_pack_tag 'application', 'data-turbolinks-track': 'reload'`

### Turbolinks and Rails UJS

如果仍然想用 `Turbolinks` 和 `Rails UJS` 的话也很容易。以前用 asset pipeline 的时候，在 `assets/javascript.js` 里面 `//= require` 一下就行，但是用 Webpacker 的话就得按照前端开发那套流程来做了。

首先得安装一下 `yarn add rails-ujs turbolinks`。然后在你的入口文件中引入它们。拿上面的例子来说就是 `app/javascript/blog/index.js`:

```javascript
import Rails from 'rails-ujs';
import Turbolinks from 'turbolinks';

Rails.start();
Turbolinks.start();
```

### 开发环境测试

一个窗口开 `rails server`，另一个窗口开 `bin/webpack-dev-server` 该命令会监控你的文件修改，在需要的时候自动编译并把编译结果推送到浏览器。

不然的话，你就需要每次自己手动执行 `webpack` 或 `rails assets:precompile` 来编译。

### Environment Variables

可以通过 `process.env` 来访问环境变量。 `export const STRIPE_API_KEY = process.env.STRIPE_API_KEY;`

### Stylesheets

使用 CSS 和 SCSS 也很简单，还是用上面的例子，样式文件入口在 `app/javascript/blog/styles/app.scss`，把你需要的其它文件放在 `styles` 目录下。然后在 `app/javascript/packs/application.js` 的文件中增加

```javascript
import './styles/app.scss';
```

需要注意的是，如果你要引入 `node_modules` 内的 CSS 文件，需要用 `~` 字符：

```javascript
@import '~bootstrap/scss/bootstrap';
```

### Images

跟 CSS 差不多，在使用前需要在 JavaScript 中先引入它们。比如图片在 `app/javascript/blog/images` 中，需要在 `app/javascript/blog/images/index.js` 添加：

```javascript
import './logo.svg';
import './menu-open.svg';
import './menu-close.svg';
```

之后就可以用 `asset_pack_path` 方法来使用这些图片了，比如 `= image_tag asset_pack_path('logo.svg')`。

## 相关

* [Replacing the Rails Asset Pipeline with Webpack and Yarn](http://samuelmullen.com/articles/replacing-the-rails-asset-pipeline-with-webpack-and-yarn/)
* [Replacing Rails Asset Pipeline with Webpacker](https://www.neontsunami.com/posts/replacing-rails-asset-pipeline-with-webpacker)
* [Replacing the asset pipeline with Webpack 2 in Rails](http://www.krisquigley.co.uk/2017/02/17/replacing-the-asset-pipeline-with-webpack-2-in-rails.html)
