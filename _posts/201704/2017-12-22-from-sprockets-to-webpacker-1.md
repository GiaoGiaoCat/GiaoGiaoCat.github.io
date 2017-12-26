---
layout: post
title:  "从 Sprockets 迁移到 Webpacker（上）"
date:   2017-12-22 11:30:00
categories: rails
tags: rails5 javascript
author: "Victor"
---

## 为何要迁移

当年 Asset Pipeline 在 Rails 3.1 出现的时候确实很惊艳，但是近年来前端社区越来越火热，在大家的集体努力下有很多不错的改进和特性。而 Asset Pipeline 也有几个问题：

* Sprockets is too slow, i.e., in development, we don't want to run JavaScript compilation through our Rails process
* To adopt ES6 syntax and Sprockets support for ES6 is experimental
* For advanced features not available in Sprockets (or without extra effort), i.e., modularity, tree-shaking, live-reload, configurable source-maps, etc.

而且用 Rails 的开发者最好还是的跟着大部队走，既然 Rails 官方团队都搞出 Webpacker 了，我觉得无脑跟进就行。

## Webpack, the Rails Way

Webpacker 是在 Rails 中使用 Webpack 的方案。它主要有下面几个好处。

1. Webpack 很好，但是从头去配置一份适合你项目的 webpack 配置文件还是挺麻烦的，webpacker 将项目中常用的 loader 和 plugin 打包成了默认配置，开箱即用。
2. Webpacker 是 Webpack 应用和 Rails 应用之间的桥梁。你可以使用 `javascript_pack_tag` 在 Rails 的视图中生成 `<script>` 标签来引入 webpack 的资源。
3. 根据 Webpacker 的约定，Webpack 从 `app/javascript` 和 `node_modules`（由 yarn 安装）读取源文件来构建 JavaScript。Webpack 会把 `app/javascript/packs` 根目录下的每一个文件当作一个入口文件。

开发环境下，`rake assets:precompile` 会分别编译 Sprockets 和 Webpack 的文件。默认的产品环境，每个 Webpack 的入口文件会编译成一个输出文件放在 `public/packs` 目录下，而 Sprockets 的编译文件放在 `public/assets` 目录下。Webpack 在 `public/packs` 下生成一个 `manifest.json` 用来映射 asset names 和 locations 的对应关系。Rails will read the manifest to determine the urls for Webpack assets.

开发环境比较好的选择是独立运行 Webpack Dev Server，来监控 JavaScript 文件的改变，自动编译和刷新浏览器。更棒的是，开发环境 Webpacker 插入了一个 Rails middleware 来代理 webpack 的 asset 请求到 Webpack Dev Server。

## Making a plan

Webpacker 让 Webpack 和 Sprockets 可以并存。我们可以让 Webpack 编译 some_module.js 而让 Sprockets 编译 another_module.js。也就是说我们可以在不破坏网站的前提下，一步一步的升级我们的项目，以渐进的方式迁移到 Webpack。

简述一下迁移的大体步骤：

1. 准备阶段
  1. Add Webpacker and setup dependencies in development and remote servers (upgrade node.js, install yarn)
  2. Deploy a small Webpack bundle (with no critical code) to iron out deployment concerns, including Nginx and capistrano configuration
2. 迁移阶段
  1. Move our third-party dependencies from Rails asset gems to NPM packages via Webpack
  2. Move our application code to Webpack
  3. Modify Webpack configuration as needed to support new dependencies
3. 清理阶段
  1. Remove Rails assets gems and redundant Sprockets configuration
  2. Optimize our Webpack bundles

当然这个方式也有其缺点：

1. We needed to figure out how to reference modules across two scopes
2. We had a large suite of JavaScript unit tests to support in two separate testing environments
3. We assumed global variables our in Sprockets-based JavaScript, so any module bundled by Webpack would need to be exposed to the global scope somehow
4. We had a learning curve with Webpack such that simply moving a dependency from a Sprockets bundle to a Webpack bundle was not always straightforward

## Setting up Webpack entries

我们团队的项目有两个 JavaScript 入口文件，`vendor.js` 和 `application.js`。`vendor` 用来放第三方的类库，比如 `jQuery, knockout.js, and lodash`。`application` 用来放经常修改的文件，比如比较小的三方插件和应用需要的代码。

看起来我们的代码结构如下：

```
app/assets/javascript
|-- vendor.js
|-- application.js
|-- some_module.js
|-- another_module.js
```

```html
<!-- application.html.erb -->

<html>
    <body>
        <!-- ... -->

        <%= javascript_include_tag 'vendor' %>
        <%= javascript_include_tag 'application' %>
    </body>
</html>
```

第一步要做的就是在 packs 目录下创建对应的 `vendor.js` 和 `application.js`，并在 HTML 中添加引入。

```
app/assets/javascript
|-- vendor.js
|-- application.js
|-- another_module.js
|-- some_module.js
app/javascript
|-- packs
    |-- vendor.js
    |-- application.js
```

```html
<!-- application.html.erb -->

<html>
    <body>
        <!-- ... -->
        <%= javascript_pack_tag 'vendor' %>
        <%= javascript_include_tag 'vendor' %>

        <%= javascript_pack_tag 'application' %>
        <%= javascript_include_tag 'application' %>
    </body>
</html>
```

接下来把 `app/assets/javascripts` 中的 js module 文件，分别移动到 `app/javascript`，并把语法升级到 ES6。

```
app/assets/javascript
|-- vendor.js
|-- application.js
|-- another_module.js
app/javascript
|-- some_module.js
|-- packs
    |-- vendor.js
    |-- application.js
```

## Maintaining backwards compatibility 保持向后兼容

当我们把三方类库和组件从 asset pipeline 移动到 Webpack 后，需要确保向后兼容，也就是说迁移的代码可以和未迁移的代码协同工作。

以本文中的项目为例，Rails view、Knockout templates、遗留的 JavaScript 代码就需要依赖一些全局的 `jQuery, lodash, knockout` 方法才能正确生效。为了降低风险，暂时不会改变它。

说到这里需要先了解 Sprockets 和 Webpack 是两种完全不同的打包 JavaScript 的范式。Sprockets 把所有的 JavaScript 打包进一个全局作用域，而 Webpack 在运行时为每个 JavaScript 模块提供单独的作用域，所以模块之间的访问必须要先 `import`。默认配置下，这些模块不会暴露在全局作用域中。如果想了解更多，原文中的 **What problem does Webpack solve?** 部分给出了一些相关文章。

作为本次迁移的目的之一。我们不希望 Webpack 模块依赖全局变量。这意味着在 Webpack 中不能引用 Sprockets 编译的代码。因此，在迁移单独的模块文件 `some_module.js` 时，需要注意下面两个问题：

1. can we import all third party dependencies of some_module.js from Webpack?
2. can we import all application dependencies of some_module.js from Webpack?

事实上，我们不得不在某些情况下妥协。即便大部分第三方的 JavaScript 代码可以移动到 yarn 的 node_modules 中，还是有一部分代码是通过浏览器加载 `<script>` 标签的方式调用其 API。比如，Google Analytics 就不太好编译到 Webpack 中来。

## Migrating a Javascript Module

我们原来的代码需要访问一个全局变量 `window.App`：

```javascript
// app/assets/javascripts/some_module.js

App = App || {};

App.SomeModule = (function() {
  someMethod: function() {
    var timestamp = moment();
    return App.AnotherModule.method(timestamp);
  }
}();
```

把代码移动到 Webpack 时，需要改造成符合 ES6 规范并且改用 `import` 引用的方式：

```javascript
// app/javascript/some_module.js

import moment from 'moment';
import AnotherModule from './another_module';

const SomeModule = {
  someMethod() {
    const timestamp = moment();
    return AnotherModule.method(timestamp);
  }
};

export default SomeModule;
```

这里有个问题。一旦 `SomeModule` 移动到 Webpack，就不能通过全局作用域的方式访问 `App` 的属性了， 在 Sprockets 中调用 `App.SomeModule` 会报 `undefined` 的错。为了保持向后兼容，我们需要找到一种可以使 Webpack 和 Sprockets 均可使用 `SomeModule` 的方法。

意思是下面两个得都好使：

```javascript
// `SomeModule` could be available as an import in Webpack
import SomeModule from '../some_module';

SomeModule.someMethod();

// global scope as a property the global `App` instance
App.SomeModule.someMethod();
```

还好 ES6 module 允许我们这么做。

## Exporting from Webpack

Webpack 提供的 `library authors` 正好满足了我们的需求。我们通过配置 [output-library](https://webpack.js.org/configuration/output/#output-library) 让 Webpack 接受一个 `scope` 并将变量输出到该作用域，在本例中就是 `window`。也就是说我们把 Webpack modules 打包到一个 library 供 Sprockets 使用。

下面是我们的配置文件：

```javascript
// config/webpack/shared.js

output: {
  // Makes exports from entry packs available to global scope, e.g.
  library: ['Packs', '[name]'],
  libraryTarget: 'var'
},

// ...
```

上面这份配置的意思是，Webpack 会输出一个名为 `Packes` 的模块到全局作用域。`Packs` 变量的属性可以通过名称的方式访问各个入口。本例中，意思是 Webpack 导出了 `Packs.vendor` 和 `Packs.application` 属性。

接下来还需要把模块添加到 library 中，如下：

```javascript
// app/javascript/packs/application.js

export { default as SomeModule } from "./some_module";
```

现在 `Packs.application` 模块就有了 `SomeModule` 属性：`Packs.application.SomeModule`。

虽然上面我们已经搞定了 `Packs` 全局变量，还得添加一点代码把 `Packs` 模块添加到 `App` 命名空间才行：

```javascript
// app/assets/javascripts/application.js

App = App || {};
_.assign(App, Packs.application);
```

哈哈，现在就可以在 Sprockets 中通过 `App.SomeModule` 来使用 Webpack 编译的模块了。

## Resolving application modules

为了简单，我们希望导入模块的时候可以直接使用 `import 'some_module'` 而不是 `import '../some_module'`。为此需要在 `.babelrc` 添加一个别名。Webpacker 为 Babel 单独创建了一个配置文件叫 `.babelrc`。我们添加了 `babel-plugin-module-resolver` 插件并修改 `.babelrc` 中的插件部分：

```javascript
// .babelrc

{
  // ...
  "plugins": [
    // ...
    ["module-resolver", { "root": ["./app/javascript"], "alias": {} }]
  ],
}
```

## Extending the Webpack configuration

虽然 Webpacker 的默认配置已经足够起步阶段，但是每个项目都需要根据不同情况做自己的调整。这一部分可以直接参看 [Webpaker Configuration文档](https://github.com/rails/webpacker/blob/master/docs/webpack.md)。

## Importing libraries and global scope

## Discovering Webpack chunks

## Extracting common chunks

## 参考

* [Introducing Webpacker](https://medium.com/statuscode/introducing-webpacker-7136d66cddfb)
* [How we switched from Sprockets to Webpack](https://rossta.net/blog/from-sprockets-to-webpack.html)
* [Yarn Packages](https://yarnpkg.com/en/packages)
