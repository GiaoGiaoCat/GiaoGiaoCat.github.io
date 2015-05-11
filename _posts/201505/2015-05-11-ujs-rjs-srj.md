---
layout: post
title:  "RJS, UJS 和 SRJ 的区别"
date:   2015-05-11 16:00:00
categories: rails
tags: enumerable
author: "Victor"
---

## RJS - Remote JavaScript

RJS 出现在 Rails1.1 之后，它基于 Prototype 和 Scriptaculous。当时 Rails 团队想利用 RJS 让开发者更容易地从单个 Ajax 请示内来更新多个页面元素并创建复杂的视觉效果。

它返回的 JavaScript 完全由 Ruby 编写，下面是一个典型的 RJS 模板

```ruby
page.insert_html :bottom, 'thoughts', :partial => 'thought'
page.visual_effect :highlight, 'thoughts'
page.form.reset 'thought-form'
```

当时 Rails 为了让开发者方便的使用 RJS 提供了不少内置的辅助方法 `link_to_remote, form_remote_tag, submit_to_remote, in_place_editor_field` 等等。

后来，各种原因让 Rails 团队放弃了 RJS 转用 UJS。我猜主要的原因是 jQuery 的流行度取代了 Prototype，并且 RJS 模板的写法太死板，你想要写点原生的 JavaScript 都不行。

## UJS - Unobtrusive Javascript

UJS 的目的是从 HTML 中分离出 JavaScript 代码。它是从 Rails 3 开始出现的，集成进入了 [jquery-rails](https://github.com/rails/jquery-ujs) 这个 gem，成为了 Rails 的标配。

它的思想和 RJS 并无不同，只是模板可以使用原生 JavaScript 代码从而让开发者的定制性更高。

### 功能

* 为各种操作增加确认对话框
* 让普通超链接支持非 GET 请求
* 让表单和超链接以异步的 Ajax 方式来提交数据
* 已经提交按钮会自动变成禁用表单提交，以防止重复点击造成多次提交

### 用法

过去我们要为链接增加一个确认对话框，可能会这么写

```html
<!DOCTYPE html>
<html>
  <head>
    <title>UJS Example</title>
    <script src="jquery.min.js" type="text/javascript" charset="utf-8"></script>
    <script type="text/javascript" charset="utf-8">
      $(function() {
        $("#alert").click(function() {
          alert(this.getAttribute("data-message"));
          return false;
        })
      })
    </script>
  </head>
  <body>
    <h1><a href="#" id="alert" data-message="Hello UJS!">Click Here</a></h1>
  </body>
</html>
```

当我们引入了 jquery-rails 之后，大家都会写出下面的代码，类似于过去的 `link_to_remote`

```ruby
<%= link_to 'Destroy', @product, confirm: 'Are you sure?', method: :delete %>
```

它会生成

```html
<a href='/products/8' data-confirm='Are you sure?' data-method='delete' rel='nofollow'>Destroy</a>
```

没错 Rails 框架提供的辅助方法会通过在 HTML 的 data 标记上附加一些数据，来实现 UJS 的功能。

下面是一个比较常见的例子

```erb
<!-- products/index.html.erb -->
<% form_tag products_path, :method => :get, :remote => true do %>
  <p>
    <%= text_field_tag :search, params[:search] %>
    <%= submit_tag "Search", :name => nil %>
  </p>
<% end %>

<div id="products">
  <%= render @products %>
</div>
```

```erb
<!-- products/index.js.erb -->
$("products").update("<%= escape_javascript(render(@products)) %>");
```

## SJR - Server-generated JavaScript Responses

Basecamp 的绝大多数 Ajax 操作是由服务器端生成并返回 JavaScript (SJR)，它的工作流程如下：

1. 通过 XMLHttpRequest-powered 来提交表单
2. 服务器端创建或更新一个 model 对象
3. 服务器端生成并返回 JavaScript，它包含更新该 model 的 HTML 模板
4. 客户端解析这段由服务器返回的 JavaScript, 然后更新 DOM

这个简单的模式有许多重要的好处。

### 好处 #1: 重用模板且不牺牲性能

可以在第一次渲染 model 和后续更新中重复使用同一模板。例如在 Rails 项目中, 你可以在这两种情况下都使用 messages/message 这个局部模板。

如果你只返回 JSON, 为了显示 messages 你需要执行两次动作(一次是服务器端的 first-response, 一次是客户端的 subsequent-updates) —— 除非你正在做一个单页面的 JavaScript 应用程序，甚至第一次响应也是 JSON/client-side 生成并完成的。

使用第2种方法可能非常缓慢, 因为你不能显示任何东西, 直到你的整个 JavaScript 库都加载完毕, 然后客户端生成的模板。(这是Twitter最初使用的方法，后来抛弃了)。但至少在某些情况这是一个合理的选择, 且不需要重复模板。

### 好处 #2: 客户端做更少的运算

嵌入 HTML 模板的 JavaScript 的响应时间可能略大于纯 JSON 的响应时间(当你使用 gzip 压缩时, 这种差距通常可以忽略不计)。它不需要客户端的运算来更新。

考虑到这些模板的复杂性和客户端的计算能力, 且服务器端生成模板通常可以缓存并在多用户间共享(参考 Russian Doll caching)。这意味着，从 end-to-end 的角度发送 JavaScript+HTML 有可能比 JSON 配合客户端生成模板更快。

### 好处 #3: 浅显易懂的执行流程

SJR 是一个很容易遵循的执行流程。请求机制是一个标准化的 helper 逻辑, 例如 ```form_for @post, remote: true```。 没有必要为每一个请求定义 action 逻辑。控制器执行逻辑，然后渲染部分视图生成响应，以完全相同的方式呈现一个完整的视图, 只是模板变成 JavaScript, 而不是 HTML。

完整的例子

0) First-use of the message template.

```erb
<h1>All messages:</h1>
<%# renders messages/_message.html.erb %>
<%= render @messages %>
```

1) Form submitting via Ajax.

```erb
<% form_for @project.messages.new, remote: true do |form| %>
  ...
  <%= form.submit "Send message" %>
<% end %>
```

2) Server creates the model object.

```ruby
class MessagesController < ActionController::Base
  def create
    @message = @project.messages.create!(message_params)

    respond_to do |format|
      format.html { redirect_to @message } # no js fallback
      format.js   # just renders messages/create.js.erb
    end
  end
end
```

3) Server generates a JavaScript response with the HTML embedded.

```erb
<%# renders messages/_message.html.erb %>
$('#messages').prepend('<%=j render @message %>');
$('#<%= dom_id @message %>').highlight();
```

最后一步反应是由 form_for 生成的 XMLHttpRequest-powered 自动处理, 视图更新新消息, 新消息通过 JS / CSS 动画实现高亮效果。

### 让你海阔天空的 RJS

当我们第一次开始使用 SJR 时, 我们使用一种叫 RJS 的编译器, 它允许你用 Ruby 编写模板，并转换成 JavaScript。 RJS 是一个屌丝版本的 CoffeeScript 你也可以了解一下 [Opalr](http://opalrb.org/), RJS 让许多人离开了 SJR 模式。

现在我们不再使用 RJS, 但是我们仍然坚定的推荐 SJR。

这并不意味着由服务器端生成 JSON 然后由客户端生成 views 的情况不再需要。如下场景我们会用到这个模式：UI 保真度非常高并且大量的视图状态需要维护。例如日历。当你遇到这个情况时，我们建议使用 [Eco template system](https://github.com/sstephenson/eco)(think ERB for CoffeeScript)

如果你的 web application 全都是 high-fidelity UI, 它完全合适这种路线。当然自己选择的路跪着也要走完！爷们不哭，站起来撸。但是如果你的程序更像 Basecamp 或者 Github 或者绝大多数基于文档的 web 站点, 那么你真的应该使用 SJR。

最后，DHH大大认为 [Russian Doll-caching](http://37signals.com/svn/posts/3112-how-basecamp-next-got-to-be-so-damn-fast-without-using-much-client-side-ui), [Turbolinks](https://github.com/rails/turbolinks) 和 SJR。绝对是一个牛X的技术组合，能够让你高大强的为 web 应用写出漂亮的代码。颤抖吧凡人。

## 总结

* RJS - sends a js file from server-side to replace the part of html, but the js is written by all ruby
* Pjax - sends a js file for rendering partial page, including page state
* Turbolink - uses a js control all rendering page except template part, it's added in Rails 4
* SJR - sends a js file from server-side to trigger js event, it's written in javascript and DHH recommends this way

## 相关阅读

* [Rails 3, jQuery & UJS](http://chrissloan.info/blog/rails_3_jquery_ujs/)
* [Unobtrusive Javascript](http://railscasts.com/episodes/205-unobtrusive-javascript)
* [Design patterns and AJAX in Rails - From DHH's podcast](http://terratakashi.logdown.com/posts/175488-design-patterns-and-ajax-in-rails-from-dhhs-podcast)
* [The security risks of RJS/SJR](https://github.com/jcoglan/unsafe_sjr/blob/master/README.md)
* [A Tour of Rails’ jQuery UJS](https://robots.thoughtbot.com/a-tour-of-rails-jquery-ujs)
* [Server-generated JavaScript Responses](https://signalvnoise.com/posts/3697-server-generated-javascript-responses)
