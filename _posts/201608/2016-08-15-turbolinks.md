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

## 使用 Turbolinks 进行页面导航

Turbolinks 会接管同一域名下的所有 `<a href>` 的点击事件。当你点击一个符合该条件的链接，Turbolinks 会阻止浏览器默认的跳转行为。而使用 [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) 修改浏览器的 URL，使用 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 获取新页面内容，然后渲染 HTML 响应。

在渲染的过程中，Turbolinks 完全替换掉 `<body>` 内容，合并 `<head>` 中的内容。JavaScript 的 `window` 和 `document` 对象，以及 HTML 的 `<html>` 元素，会从一个页面携带到下一个页面。

### 所有的跳转都是 Visit

Turbolinks 模式下的页面跳转就像通过一个 action 来访问（visit）一个 URL。

访问（visit） 表示一个页面跳转从点击到渲染完毕的生命周期。包含修改浏览器历史，发起网络请求，从缓存中获取页面数据，渲染最后的响应，更新滚动条。

有两个类型的visit:

* Application Visits: `advance` 和 `replace`
* Restoration Visits: `restore`

### Application Visits

当点击链接或者调用 `Turbolinks.visit(location)` 方法的时候，就会触发 Application Visits 模式。该模式总会发起网络请求，当取回数据的时候，Turbolinks 渲染  HTML 并完成整个 visit。

如果可能，当 visit 开始的时候 Turbolinks 会立即从缓存中取出内容并渲染一个预览页面。这改善了你访问同一个页面的速度。

如果请求的 URL 包含一个锚链接，Turbolinks 会自动滚动到该锚节点。

visit 的动作决定了 Application visits 如何修改浏览器的访问历史记录。

默认的 visit 动作是 `advance`，这种情况下 Turbolinks 使用 [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API#Adding_and_modifying_history_entries) 推送一条新记录到浏览器的访问历史中。

iOS 应用会推送一个新的 view controller 到导航栈中，Android 会推送一个新的 activity 到回退栈。

如果不想在栈中添加一个新的历史记录，`replace` 动作使用 [history.replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) 干掉最后的一条访问记录，并用新记录来替换它。

你可以使用下面的方法来触发 `replace` 动作。

```html
<a href="/edit" data-turbolinks-action="replace">Edit</a>
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

当 visit 开始发起的时候，使用 `event.data.url` 或 jQuery 的 `$event.originalEvent.data.url` 访问 URL，`turbolinks:before-visit` 事件会得到通知。所以你只需要监听这个事件，并调用 `event.preventDefault()` 来取消请求。

Restoration visits 无法被取消，它也不会触发 `turbolinks:before-visit`。也就是说你用浏览器的前进和后退按钮是无法触发 `turbolinks:before-visit` 的。

### Disabling Turbolinks on Specific Links

可以通过如下方法，禁用或启用 Turbolinks

```html
<a href="/" data-turbolinks="false">Disabled</a>
<a href="/" data-turbolinks="true">Enabled</a>
```

## 创建你的 Turbolinks 应用

采用 Turbolinks 技术的应用访问速度很快，因为当你点击链接的时候并没有重新加载页面。应用看起来像一个持久化运行的应用。这让我们不得不重新思考项目中的 JavaScript 架构。

尤其要注意的是，每次进入一个新页面不会再重置环境变量。JavaScript 的 `window` 和 `document` 对象会在页面间携带它们的状态。你在内存中留下的任何对象都会一直待在内存里。

不用担心，只要注意一些细节。你的应用就能很优雅的处理掉这些问题，而不需要跟 Turbolinks 耦合在一起。

### Running JavaScript When a Page Loads
过去我们使用 `window.onload`，`DOMContentLoaded` 或 jQuery 的 `ready` 事件来添加一个行为。而 Turbolinks 应用，只会在第一次初始化页面的时候执行这些函数，也就是你点击链接进入其它页面的时候，所有这些事件下面注册的行为都不会再次执行。

我们应该改用 `turbolinks:load`

```javascript
document.addEventListener("turbolinks:load", function() {
  // ...
})
```

另外，不要在页面的 `body` 中使用 `turbolinks:load` 来注册监听事件，而是在 `document` 和 `window` 上利用 [event delegation](https://learn.jquery.com/events/event-delegation/) 注册监听。

### Working with Script Elements
在首次初始化页面的时候，浏览器会自动加载和计算 `<script>` 元素。在进入新页面的时候，Turbolinks 会去新页面的 `<head>` 区域寻找与当前页面不同的 `<script>` 元素，加载并运行完毕。所以，你可以在这里追加你所需要的 JavaScript 文件。

Turbolinks 当进入一个新页面的时候会运行该页面 `<body>` 中的 `<script>`。你可以在这里放一些只在当前页面有效的代码，比如设置当前页面的 JavaScript 状态或 bootstrap 客户端模型。

在 `<script>` 上增加 `data-turbolinks-eval="false"` 会阻止 Turbolinks 运行该脚本。但是，这个方法无法阻止浏览器在第一次加载页面的时候运行这个脚本。

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

当从页面显示的是缓存中的内容时，Turbolinks 会在 `<html>` 元素上增加一个 `data-turbolinks-preview` 属性。你可以根据是否存在这个属性来检查你的内容是不是缓存的。

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

### Making Transformations Idempotent(幂等)

有时，从服务端获取 HTML 后我们想在客户端做一些处理。比如，浏览器知道用户的时区，客户端可以根据日期把数据分组显示。

假设有一组元素，它们的 `data-timestamp` 属性用 UTC 方式标注了该对象在数据库中的创建时间。现在你想用 JavaScript 函数来查询这些元素，把时间戳改成本地时间，并在每个元素前增加一个元素显示日期。

试想，如果这件事你想在 `turbolinks:load` 里面做的话会怎样。当你进入页面，你的函数插入了日期元素。你点击链接进入其它页面，Turbolinks 把刚才的页面存入缓存。现在点击浏览器的后退按钮，`turbolinks:load` 会再次执行，所有元素前面又增加了第二组日期元素。

为了避免这个问题的，我们需要让所有页面载入的函数操作均幂等。幂等的函数可以安全的多次执行，其不会破坏程序初始化结果。

一个让过场函数幂等的技巧是给每一个需要处理的元素上面添加一个 `data` 属性，然后观察这个值，以便判断是否已经在该元素上执行过函数操作。因为当 Turbolinks 从缓存中 restores 页面的时候, 这些属性值仍然存在，所以我们可以通过在函数中检测这些属性来确定哪些元素已经被处理过。

另一个更好的办法是检测函数自身。也就是上面日期分组那个例子，当我们要插入一个新的日期之前，先检测它是否已经存在。这一技巧通常用来处理在页面中插入新元素的情况。

### Responding to Page Updates

Turbolinks 可能并不是你应用中唯一修改页面内容的代码。Ajax 请求、WebSocket 连接或其它客户端渲染操作都会插入新的 HTML，这些内容也需要像新页面载入的时候被初始化。

你可以处理上面提到的这些更新过程，包括通过 Turbolinks 加载页面而引起的更新。in a single place with the precise lifecycle callbacks provided by [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) and [Custom Elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements)。

利用这些 APIs 可以在页面中加入元素或删除元素时增加回调。使用回调，当我们发现页面出现元素的时候来实现渲染、注册或拆除行为，而不用管这些元素是通过什么手段添加到页面中的。

通过 MutationObserver, Custom Elements, 和 idempotent transformations 带来的优势，只需要处理很小的改动就可以让应用程序兼容 Turbolinks 事件。

### Persisting Elements Across Page Loads
