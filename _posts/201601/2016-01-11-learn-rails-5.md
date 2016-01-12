---
layout: post
title:  "测试 Rails 5 的新功能"
date:   2016-01-11 14:30:00
categories: rails
tags: rails5 learningnote
author: "Victor"
---

## 安装

```bash
rvm use 2.2.2@rails5
cd ~/Documents/Githubs
git clone --depth 1 git@github.com:rails/rails.git
cd rails
gem install bundler
bundle
railties/exe/rails new --edge fiverails
mv fiverails ~/Documents/Githubs/
cd ~/Documents/Githubs/fiverails
bundle
bundle exec rails s
```

如果遇到 `gem install pg -v '0.18.4'` 无法安装的问题，需要先安装 postgresql：

```bash
brew install postgresql
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```

## Rails 5 的变化

### Upcoming Deprecations

1. `to_xml` 方法从核心代码中删除了，现在 `object.to_xml` 会直接抛错
2. `render nothing: true` 将从 Rails 5.1 中删除，所以现在也不推荐使用了。新的写法是 `head: ok`具体 [参考这里](http://stackoverflow.com/questions/4632271/render-nothing-true-returns-empty-plaintext-file)
3. `before_filter` 将从 Rails 5.1 中删除，一律改用 `before_action`

```ruby
# rails 4.x
def nothing
  render nothing: true
end

# rails 5.1
def nothing
  head :ok
end
```

### Upcoming Changes to Existing Features

* Rails 5 把字符串 "1" 和布尔值 true 都可以认为 accept is true。注意：整形 1 仍然被认为是 false。参考 [`validates_acceptance_of` Accepts true](https://github.com/rails/rails/pull/18439)
* `belongs_to` 外键约束默认不能为空，参考 [`belongs_to` is Now Required in ActiveRecord by Default](https://github.com/rails/rails/pull/18937)

```ruby
g model post
g model comment post:belongs_to
```

```ruby
class CreateComments < ActiveRecord::Migration[5.0]
  def change
    create_table :comments do |t|
      t.belongs_to :post, index: true, foreign_key: true

      t.timestamps
    end
  end
end
```

从这里我们可以看到，Rails 在创建数据库结构的时候，越来越注意外键约束的问题了。

```ruby
g model comment post:belongs_to:optional
```

```ruby
class CreateComments < ActiveRecord::Migration[5.0]
  def change
    create_table :comments do |t|
      t.belongs_to :post, index: true, foreign_key: true optional: true

      t.timestamps
    end
  end
end
```

### New ActiveRecord Features

`or` 方法，可以参考另外一篇转载 [What's new in Rails 5](/rails/rails-5/)

可以看到 Rails 5 改进了验证错误提示，增加了 `details` 变量。

```ruby
Comment.new.tap(&:valid?).errors

# Rails 5.x
#=> #<ActiveModel::Errors:0x007f993429c618 @base=#<Comment id: nil, post_id: nil, created_at: nil, updated_at: nil>, @messages={:name=>["can't be blank"], :post=>["must exist"]}, @details={:post=>[{:error=>:blank}]}>

# Rails 4.x
#=> #<ActiveModel::Errors:0x0000010e380bf8 @base=#<Comment id: nil, post_id: nil, created_at: nil, updated_at: nil>, @messages={:name=>["can't be blank"], :post=>["must exist"]}>
```

Rails 5 提供了更安全的 token 机制，数据库结构默认增加了索引以及唯一性约束。同时添加了动态方法 `regenerate_xxx`

```ruby
bin/rails g model MyModel my_token:token

class CreateMyModels < ActiveRecord::Migration[5.0]
  def change
    create_table :my_models do |t|
      t.string :my_token

      t.timestamps
    end
    add_index :my_models, :my_token, unique: true
  end
end

class MyModel < ApplicationRecord
  has_secure_token :my_token
end
```

```ruby
m = MyModel.new
m.regenerate_my_token
```

```
(0.1ms)  begin transaction
SQL (0.4ms)  INSERT INTO "my_models" ("my_token", "created_at", "updated_at") VALUES (?, ?, ?)  [["my_token", "4HedGeNQG8JRQ4NsTFywvxte"], ["created_at", 2016-01-12 07:29:26 UTC], ["updated_at", 2016-01-12 07:29:26 UTC]]
(3.4ms)  commit transaction
```

我们不用再关心手动创建 token，Rails 会帮我们处理好一切。

```ruby
MyModel.create
=> #<MyModel id: 2, my_token: "GqzBgs8NAUcwX1FH5hEGKvE2", created_at: "2016-01-12 07:31:11", updated_at: "2016-01-12 07:31:11">
```

Rails 4.x 的时候，如果我们想让模型中的 callback 失败，只需要在方法中返回 `false` 或者 `nil`，而 Rails 5 开始需要更改为 `throw :abort`。

```ruby
class Post < ApplicationRecord
  before_save :do

  def do
    throw :abort
    # return false NOT works in rails 5
  end
end
```

```
Post.create #=>
(0.1ms)  begin transaction
(0.0ms)  rollback transaction
```

### Rails API

```bash
cd ~/Documents/Githubs/rails
railties/exe/rails new --edge --api fiverailsapi
mv fiverailsapi ~/Documents/Githubs/
cd ~/Documents/Githubs/fiverailsapi
bundle
bundle exec rails s
rails g scaffold project name ends_at:timestamp
```

### Turbolinks 3

我们有一个如下的界面

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-01-11-learn-rails-5/01.png)

当用户添加评论之后，我们希望把新评论追加到评论列表的末尾。利用 SRJ 我们可以这么写

```ruby
# app/views/comments/create.js.erb

$("#comments .new-comment").before("<%= escape_javascript(render(partial: 'task/comment', locals: { comment: @comment })) %>")
```

这样的缺点是我们没办法同时更新原来评论列表中的内容，比如：发布于 xxx 分钟前。

现在我们可以这么写

```ruby
# app/views/comments/create.js.erb
Turbolinks.visit("<%= project_task_path(@comment.task.project, @comment.task) %>", { change: [ 'comments'] })
```

```haml
%ul.list-unstyled#comments
  - @task.comment.each do |comment|
    = render partial: "comment", locals: { comment: comment }

  %li.new-comment
    = form_for [@task.project, @task, @task.comments.build], remote: true do |f|
      %p
        = f.text_area :body, class: "form-control", placeholder: "Post your comment here."
      %p
        = f.submit 'Post Comment', class: "btn btn-primary"
```

### ActionCable

首先做程序基础骨架

```
railties/exe/rails new campfire --edge
mv campfire ~/Documents/Githubs/
cd ~/Documents/Githubs/campfire

rails g controller rooms show
rails g model message content:text
rails db:migrate
```

```ruby
# routes.rb
root to: 'rooms#show'
```

```ruby
#app/controllers/rooms_controller.rb
def show
  @messages = Message.all
end
```

```erb
#app/views/messages/_message.html.erb
<div class="message">
  <%= message.content %>
</div>

#app/views/rooms/show.html.erb
<h1>Rooms#show</h1>
<%= render @messages %>
```

然后添加 ActionCable 并配置

```
rails g channel room speak
```

```ruby
# routes.rb
mount ActionCable.server => '/cable'
```

```coffee
#app/assets/javascripts/cable.coffee
@App ||= {}
App.cable = ActionCable.createConsumer()
```

现在启动 `rails server` 并打开首页，可以看到出现了 `<meta name="action-cable-url" content="/cable" />` 这一行：

利用 console 工具做点小测试：

```
App.cable
App.room.speak
```

继续搞起：

```coffee
#app/assets/javascripts/channels/room.coffee
App.room = App.cable.subscriptions.create "RoomChannel",
  connected: ->
    # Called when the subscription is ready for use on the server

  disconnected: ->
    # Called when the subscription has been terminated by the server

  received: (data) ->
    # Called when there's incoming data on the websocket for this channel
    alert data['message']

  speak: (message) ->
    @perform 'speak', message: message
```

```ruby
#app/channels/room_channel.rb
class RoomChannel < ApplicationCable::Channel
  def subscribed
    stream_from "room_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def speak(data)
    ActionCable.server.broadcast 'room_channel', message: data['message']
  end
end
```

利用 console 工具做点小测试 `App.room.speak('hello world')`，只要没敲错代码就能看到弹出的对话框了。

继续改进

```erb
#app/views/rooms/show.html.erb
<h1>Rooms#show</h1>

<div id="messages">
  <%= render @messages %>
</div>

<form>
  <label>Say something:</label><br>
  <input type="text" data-behavior="room_speaker">
</form>
```

```coffee
#app/assets/javascripts/channels/room.coffee
App.room = App.cable.subscriptions.create "RoomChannel",
  connected: ->
    # Called when the subscription is ready for use on the server

  disconnected: ->
    # Called when the subscription has been terminated by the server

  received: (data) ->
    # Called when there's incoming data on the websocket for this channel
    $('#messages').append data['message']

  speak: (message) ->
    @perform 'speak', message: message

$(document).on 'keypress', '[data-behavior~=room_speaker]', (event) ->
  if event.keyCode is 13 # return = send
    App.room.speak event.target.value
    event.target.value = ''
    event.preeventDefault()
```

继续改进

```
rails g job MessageBroadcast
```

```ruby
#app/channels/room_channel.rb
def speak(data)
  Message.create! content: data['message']
end

#app/models/message.rb
class Message < ApplicationRecord
  after_create_commit { MessageBroadcastJob.perform_later self }
end

#app/jobs/message_broadcast_job.rb
class MessageBroadcastJob < ApplicationJob
  queue_as :default

  def perform(message)
    ActionCable.server.broadcast 'room_channel', message: render_message(message)
  end

  private
    def render_message(message)
      ApplicationController.renderer.render(partial: 'messages/message', locals: { message: message })
    end
end
```

最后一点优化，增加缓存

```erb
#app/views/messages/_message.html.erb
<% cache message do %>
  <div class="message">
    <%= message.content %>
  </div>
<% end %>
```

### Sprockets 4

Sprockets 4 最值得介绍的改变是关于 `manifest.js`。

过去我们在 `config/initializers/assets.rb` 中指定哪些 assets 文件支持预编译。在 Sprockets 4 改用 `app/assets/config/manifest.js` 进行这项设置。事实上 Sprockets 3 就支持该功能，但是 [sprockets-rails](https://github.com/rails/sprockets-rails/blob/93a45b1c463a063ec7cf4d160107b67aa3db7a1a/lib/sprockets/railtie.rb#L77-L81) 里面进行了判断，只有当我们使用 Sprockets 4 的时候，才开启这项该功能。

```ruby
# sprockets-rails/lib/sprockets/railtie.rb

if using_sprockets4?
  config.assets.precompile  = %w( manifest.js )
else
  config.assets.precompile  = [LOOSE_APP_ASSETS, /(?:\/|\\|\A)application\.(css|js)$/]
end
```

需要注意的一点是，在 Rails 5 的正式版本中，manifest 的文件后缀可能修改成 yml，关于这一点开发团队仍在讨论中。

下面来看一个 `manifest.js` 的例子，演示如何关联 JS, CSS, fonts, images：

```javascript
// JS and CSS bundles
//
//= link_directory ../javascripts .js
//= link_directory ../stylesheets .css


// Images and fonts so that views can link to them
//
//= link_tree ../fonts
//= link_tree ../images
```

可以看到现在需要明确的指定：预编译字体和图片。

另外 `link_directory` 的意思是把这两个目录中的 `.coffee` 和 `.scss` 文件分别编译成 `.js` 和 `.css`。而 Images 和 fonts 则不需要编译。

现在我们可以从 `config/initializers/assets.rb` 中删除 `config.assets.precompile` 配置了：

```ruby
config.assets.precompile += %w(
  all.css all.js
)
```

对于小型的应用，可以直接删除 `config/initializers/assets.rb`，而像 **Basecamp** 这样的大型应用，因为有一些额外配置，所以还是需要保留它。

## 相关链接

* [Getting started with Rails 5's ActionCable and websockets](http://nithinbekal.com/posts/rails-action-cable/)
* [Action Cable – Integrated WebSockets for Rails](https://github.com/rails/rails/blob/master/actioncable/README.md)
* [Rails 5: The Sprockets 4 Manifest](http://eileencodes.com/posts/the-sprockets-4-manifest/)
