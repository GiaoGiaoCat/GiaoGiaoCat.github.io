---
layout: post
title:  "Turbolinks 5"
date:   2016-08-15 10:30:00
categories: rails
tags: rails5
author: "Victor"
---

## 简介

Turbolinks 让 Web 应用页面之间的导航变得更快。你不需要引入一个客户端的 JavaScript 框架就能得到单页应用的性能。你只需要和过去一样在服务器端渲染 HTML，然后在上面加几个超链接。当用户点击该链接的时候，Turbolinks 自动获取页面，替换掉 `<body>`，合并 `<head>`, 减少页面载入的时间。

### 功能

* 自动提高页面导航的性能。不需要指定哪个链接或哪部分页面需要被替换。
* 不需要服务端提供额外的支持。服务端仍然渲染整个 HTML 页面，而不是返回局部模板或 JSON。
* 尊重 Web 的体验。后退和刷新按钮仍然会如你期望的一样工作，Turbolinks 对搜索引擎友好。
* 支持移动端。借助 [iOS](https://github.com/turbolinks/turbolinks-ios) 和 [Android](https://github.com/turbolinks/turbolinks-android) 可以让你的应用获得近似于原生应用的体验。

### 浏览器支持

Turbolinks 的功能基于 [HTML5 History API](http://caniuse.com/#search=pushState) 和 [requestAnimationFrame](http://caniuse.com/#search=requestAnimationFrame)，所以它可以在几乎全部现代浏览器下正常工作，对于老旧的浏览器它会自动退回到标准的浏览器导航模式。

### 安装

1. 在 Gemfile 里面添加 `gem 'turbolinks', '~> 5.0.0'` 并执行 `bundle install`
2. 在 JavaScript 的 manifest 文件里面添加 `//= require turbolinks`

### 使用 Webpacker 安装

`yarn add turbolinks`

```javascript
// app/javascript/packs/application.js
import Turbolinks from 'turbolinks';
Turbolinks.start();
```

## 使用 Turbolinks 进行页面导航

Turbolinks 会接管同一域名下的所有 `<a href>` 的点击事件。当你点击一个符合该条件的链接，Turbolinks 会阻止浏览器默认的跳转行为。而使用 [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) 修改浏览器的 URL，使用 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 获取新页面内容，然后渲染 HTML 响应。

在渲染的过程中，Turbolinks 完全替换掉 `<body>` 内容，合并 `<head>` 中的内容。JavaScript 的 `window` 和 `document` 对象，以及 HTML 的 `<html>` 元素，会从一个页面携带到下一个页面。

### 所有的跳转都是 Visit

Turbolinks 模式下的页面跳转就像通过一个 action 来访问（visit）一个 URL。

访问（Visit） 表示一个页面跳转从点击到渲染完毕的生命周期。包含修改浏览器历史，发起网络请求，从缓存中获取页面数据，渲染最后的响应，更新滚动条。

有两个类型的 Visit:

* Application Visits: `advance` 和 `replace`
* Restoration Visits: `restore`

### Application Visits

当点击链接或者调用 `Turbolinks.visit(location)` 方法的时候，就会触发 Application Visits 模式。该模式总会发起网络请求，当取回数据的时候，Turbolinks 渲染  HTML 并完成整个 visit。

如果可能，当 visit 开始的时候 Turbolinks 会立即从缓存中取出内容并渲染一个预览页面。这改善了你访问同一个页面的速度。

如果请求的 URL 包含一个锚链接，Turbolinks 会自动滚动到该锚节点。

visit 的动作决定了 Application visits 如何修改浏览器的访问历史记录。

默认的 visit 动作是 `advance`，这种情况下 Turbolinks 使用 [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API#Adding_and_modifying_history_entries) 推送一条新记录到浏览器的访问历史中。

使用 Turbolinks 的 iOS 应用会推送一个新的 view controller 到导航栈中，Android 会推送一个新的 activity 到回退栈。

如果不想在栈中添加一个新的历史记录，`replace` 动作使用 [history.replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) 干掉最后的一条访问记录，并用新记录来替换它。

你可以使用下面的方法来触发 `replace` 动作。

```html
<a href="/edit" data-turbolinks-action="replace">Edit</a>
```

或者

```javascript
Turbolinks.visit("/edit", { action: "replace" })
```

这种情况下，iOS 会干掉最顶层的 view controller 再推送一个新的 view controller 进入导航栈，**并且没有动画效果**。

### Restoration Visits

当使用浏览器的后退和前进按钮时，Turbolinks 会启用 restoration visit 模式。iOS 或 Android 应用的后退动作也会触发这个模式。

如果可能，Turbolinks 会从缓存中取出页面且 **无需** 发起网络请求。否则，它会从网上获取最新的数据。下面会详细介绍缓存部分。

Turbolinks 会储存离开页面时的滚动条位置，当通过 restoration visits 模式返回这个页面的时候会自动定位到该位置。

Restoration visits 拥有 restore 动作，你无需关心它，Turbolinks 会在内部使用该动作。你无法通过外部链接或 `Turbolinks.visit` 调用 restore 动作。

### Canceling Visits Before They Start

不论是点击一个连接还是通过 `Turbolinks.visit` 发起的请求，都可以被取消掉。

当 visit 开始发起的时候，使用 `event.data.url` 或 jQuery 的 `$event.originalEvent.data.url` 可以获得请求的地址，`turbolinks:before-visit` 事件会收到通知。所以你只需要监听这个事件，并调用 `event.preventDefault()` 来取消请求。

Restoration visits 无法被取消，它也不会触发 `turbolinks:before-visit`。也就是说你用浏览器的前进和后退按钮是无法触发 `turbolinks:before-visit` 的。

### Disabling Turbolinks on Specific Links

可以禁用单独的链接或者批量禁用某个区块内部所有链接的 Turbolinks 功能。

```html
<a href="/" data-turbolinks="false">Disabled</a>

<div data-turbolinks="false">
  <a href="/">Disabled</a>
</div>
```

被禁用的区块中，可以为单独链接启用 Turbolinks

```html
<div data-turbolinks="false">
  <a href="/" data-turbolinks="true">Enabled</a>
</div>
```

## 使用 Turbolinks 技术来开发应用
采用 Turbolinks 技术的应用访问速度很快，因为当你点击链接的时候并没有重新加载页面。应用看起来像一个持久化运行的应用。这让我们不得不重新思考项目中的 JavaScript 架构。

尤其要注意的是，每次进入一个新页面不会再重置环境变量。JavaScript 的 `window` 和 `document` 对象会在页面间携带它们的状态。你在内存中留下的任何对象都会一直待在内存里。

不用担心，只要注意一些细节。你的应用就能很优雅的处理掉这些问题，而不需要跟 Turbolinks 耦合在一起。

### Working with Script Elements
在首次初始化页面的时候，浏览器会自动加载和计算 `<script>` 元素。在进入新页面的时候，Turbolinks 会去新页面的 `<head>` 区域寻找与当前页面不同的 `<script>` 元素，当这些代码加载并执行完毕之后，会被追加到 `<head>` 中。所以，你可以在这里追加你所需要的 JavaScript 文件。

每次进入一个页面的时候，Turbolinks 会运行该页面 `<body>` 中的 `<script>`。你可以在这里放一些只在当前页面有效的代码，比如设置当前页面的 JavaScript 状态或 bootstrap 客户端模型。

避免使用 `<body>` 中的 `<script>` 添加注册行为，当页面改变时执行一些复杂的操作的代码，而使用 `turbolinks:load` 事件去完成这些事情。

在 `<script>` 上增加 `data-turbolinks-eval="false"` 会阻止 Turbolinks 运行该脚本。但是，这个方法无法阻止浏览器在第一次加载页面的时候运行这个脚本。

### Loading Your Application’s JavaScript Bundle

确保只在 `<head>` 中通过 `<script>` 标签来加载应用的 JavaScript bundle 文件。否则 Turbolinks 会在页面每次更新的时候重载 bundle 文件。

```html
<head>
  ...
  <script src="/application-cbd3cd4.js" defer></script>
</head>
```

如果出于加载性能方便的考虑，你想按照传统的方式把 `<script>` 标签放在 `<body>` 的底部部分，那可以使用 `<script defer>` 属性，该属性已经被大部分浏览器所支持。`defer` 表示该脚本的加载和其后文档的加载是异步进行，但该脚本的执行要在所有元素解析完成之后，`DOMContentLoaded` 事件触发之前完成。

另外不要忘记，使用一个可以生成指纹的前端编译工具。这样一旦 JavaScript bundle 文件内容有变化的时候，文件名会携带新指纹。当部署新的 JavaScript bundle 文件时，可以使用 `data-turbolinks-track` 属性让页面重新加载。详情后面的 Reloading When Assets Change  部分会继续介绍。

### 理解缓存

Turbolinks 会为最近访问的页面维护一份缓存。缓存有两个目的：在 restoration visits 时，不需要网络请求就可以显示页面；在 application visits 时，通过显示一个临时页面来改善性能。

When navigating by history (via Restoration Visits), Turbolinks will restore the page from cache without loading a fresh copy from the network, if possible.

Otherwise, during standard navigation (via Application Visits), Turbolinks will immediately restore the page from cache and display it as a preview while simultaneously loading a fresh copy from the network. This gives the illusion of instantaneous page loads for frequently accessed locations.

Turbolinks 在渲染新页面前，会把当前页面的副本保存进入缓存。需要注意的是，Turbolinks 复制页面用的是 [cloneNode(true)](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)，这意味着所有这个页面注册的监听事件和关联数据都会被丢弃。

#### Preparing the Page to be Cached

如果需要在 Turbolinks 缓存文档前进行干点什么事，那么你可以监听 `turbolinks:before-cache` 事件。在这里你可以重置表单、折叠一些展开的 UI 对象、拆除一些注册的第三方小部件，这样再进入这个页面的时候，她们就能显示正常了。

```javascript
document.addEventListener("turbolinks:before-cache", function() {
  // ...
})
```

#### Detecting When a Preview is Visible

当从页面显示的是缓存中的内容时，Turbolinks 会在 `<html>` 元素上增加一个 `data-turbolinks-preview` 属性。你可以根据是否存在这个属性来检查你的内容是不是缓存的，以便决定启用还是禁用该行为。

```javascript
if (document.documentElement.hasAttribute("data-turbolinks-preview")) {
  // Turbolinks is displaying a preview
}
```

#### Opting Out of Caching

你可以通过在 `<head>` 元素上添加 `<meta name="turbolinks-cache-control">` 来控制一个页面的缓存行为。

在 application visit 模式下使用 `no-preview`，则不会从缓存中取数据渲染预览页面。标记为 `no-preview` 的页面会使用 restoration visits 模式。

如果一个页面永远不想被缓存，可以使用 `no-cache`。标记为 `no-cache` 参数的页面永远从网络取数据，即便是 restoration visits 模式。

```html
<head>
  ...
  <meta name="turbolinks-cache-control" content="no-cache">
</head>
```

如果想整站都禁用缓存，目前只能是确保所有的页面上都添加这个 `no-cache` 命令。

## Installing JavaScript Behavior

我们经常需要通过 `window.onload`, `DOMContentLoaded` 或者 jQuery 的 `ready` 事件来注册一些 JavaScript 行为。当使用 Turbolinks 的时候，这些事件只会在页面第一次加载的时候被出发，如果随后你的页面上的元素有更改，那这些事情并不会绑定在新元素上。下面介绍两个不同的方式来给 DOM 绑定 JavaScript 行为。

### Observing Navigation Events

Turbolinks 在页面跳转之间会触发一系列事件。其中最应该关注的是 `turbolinks:load`，它会在页面首次加载和每次 Turbolinks visit 后执行。

我们可以在 `DOMContentLoaded` 中观察 `turbolinks:load` 事件，以便在页面变化后添加 JavaScript 行为：

```javascript
document.addEventListener("turbolinks:load", function() {
  // ...
})
```

请记住，当这个事件被触发时，你的应用程序不会总是处于原始状态，你可能需要清理为前一页安装的行为。

另外请注意 Turbolinks 可能不是您的应用程序中页面更新的唯一方式，因此最好将初始化代码移动一个单一函数中，这样 `turbolinks:load` 或其它修改 DOM 元素的地方都可以调用该函数。

请避免使用 `turbolinks:load` 事件将其他事件侦听器直接添加到 `body` 中的元素上。 而应该使用事件委托在 `document` 或 `window` 上一次性注册事件监听器。

See the [Full List of Events](https://github.com/turbolinks/turbolinks#full-list-of-events) for more information.

### Attaching Behavior With Stimulus

新的 DOM 元素可能在任何时间出现在页面中，Ajax 请求、WebSocket 连接或其它客户端渲染操作都会插入新的 HTML，这些内容也需要像新页面载入的时候被初始化。

[Stimulus](https://github.com/stimulusjs/stimulus) 是 Turbolinks 的姐妹框架，它们使用相同的规范和生命周期。建议你用它来处理这些更新（包括 Turbolinks 页面加载的更新）。

Stimulus 需要给 HTML 添加 controller, action, 和 target attributes:

```html
<div data-controller="hello">
  <input data-target="hello.name" type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

Implement a compatible controller and Stimulus connects it automatically:

```javascript
// hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
  greet() {
    console.log(`Hello, ${this.name}!`)
  }

  get name() {
    return this.targets.find("name").value
  }
}
```

Stimulus connects and disconnects these controllers and their associated event handlers whenever the document changes using the `MutationObserver` API. As a result, it handles Turbolinks page changes the same way it handles any other type of DOM update.

### Making Transformations Idempotent(幂等)

有时，从服务端获取 HTML 后我们想在客户端做一些处理。比如，浏览器知道用户的时区，客户端可以根据日期把数据分组显示。

假设有一组元素，它们的 `data-timestamp` 属性用 UTC 方式标注了该对象在数据库中的创建时间。现在你想用 JavaScript 函数来查询这些元素，把时间戳改成本地时间，并在每个元素前增加一个元素显示日期。

试想，如果这件事你想在 `turbolinks:load` 里面做的话会怎样。当你进入页面，你的函数插入了日期元素。你点击链接进入其它页面，Turbolinks 把刚才的页面存入缓存。现在点击浏览器的后退按钮，`turbolinks:load` 会再次执行，所有元素前面又增加了第二组日期元素。

为了避免这个问题的，我们需要让所有页面载入的函数操作均幂等。幂等的函数可以安全的多次执行，其不会破坏程序初始化结果。

一个让过场函数幂等的技巧是给每一个需要处理的元素上面添加一个 `data` 属性，然后观察这个值，以便判断是否已经在该元素上执行过函数操作。因为当 Turbolinks 从缓存中 restores 页面的时候, 这些属性值仍然存在，所以我们可以通过在函数中检测这些属性来确定哪些元素已经被处理过。

另一个更好的办法是检测函数自身。也就是上面日期分组那个例子，当我们要插入一个新的日期之前，先检测它是否已经存在。这一技巧通常用来处理在页面中插入新元素的情况。

### Persisting Elements Across Page Loads

Turbolinks 允许你给某些元素标记为永久性的。`永久性元素在页面加载时保持不变。因此，在页面加载完成之后，你对这些元素进行的更改都不需要再次应用。

假设我们有一个购物车的应用。在每个页面的顶部都存在一个购物城，其中含有物品数量。这个计数器是由 JavaScript 代码动态修改的。

如果有一个用户添加一个商品到购物车中，然后在通过浏览器的回退按钮点了一次。那么 Turbolinks 会错误的从缓存中恢复前一页的状态，购物车的计数器从 1 变成 0。

你可以通过把计数器标签设置为永久性的来避免这一问题。给 HTML 设置一个 id 并用 `data-turbolinks-permanent` 来标注它为永久元素。

```html
<div id="cart-counter" data-turbolinks-permanent>1 item</div>
```

在每个渲染之前 Turbolinks 通过 id 匹配所有永久元素，并将它们从原始页面转移到新页面，同时保留它们的数据和事件监听器。

## Advanced Usage

### 加载进度条

使用 Turbolinks 方式加载页面，浏览器不会显示进度条。Turbolinks 通过插入一个基于 CSS 实现的进度条来提供加载进度反馈。

该功能默认启用。并在页面加载事件超过 500 毫秒的时候自动显示。你也可以通过 `Turbolinks.setProgressBarDelay` 方法来修改该时间。

进度条是一个 `<div>` 元素，class name 是 `turbolinks-progress-bar`。在文档首次加载的时候会应用默认样式，随后我们可以自定义规则来覆盖它的样式。

例如我们可以搞个绿色的进度条：

```css
.turbolinks-progress-bar {
  height: 5px;
  background-color: green;
}
```

也可以通过设置 `visibility` 属性为 `hidden` 来隐藏进度条。

```css
.turbolinks-progress-bar {
  visibility: hidden;
}
```
