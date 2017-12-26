---
layout: post
title:  "Webpacker 小抄"
date:   2017-12-24 10:30:00
categories: javascript
tags: rails5
author: "Victor"
---

## 起步

### 创建项目

我的推荐是去掉 Sprockets 和 Spring，用 Webpacker 来代替。

```bash
rails new look_ma_no_sprockets --skip-spring --skip-sprockets --webpack
```

使用 Rails 生成器的时候也可以选择忽略 sprockets assets 文件。

```bash
rails g controller home index --no-assets

# We'll need these in a minute:
mkdir -p app/javascript/stylesheets app/javascript/images
```

这块我有点不同的方案，后面会介绍。我觉得更好的方式是把 Webpacker 的目录指向 `app/assets`，不然满眼都是 `app/*s` 的目录，出现一个孤单的 `app/javascript` 很不和谐。

### 基本配置

在 layout 中引入 Webpacker 的 assets 文件。

```html
<!-- app/views/layouts/application.html.erb -->
<head>
  <title>LookMaNoSprockets</title>
  <%= csrf_meta_tags %>

  <%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
  <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
</head>
```

暂时还是把样式文件放在 `app/javascript/stylesheets/` 中，虽然 Webpack 的项目有很多文件目录组织方式，至少目前先扔这里吧。

```css
/* app/javascript/stylesheets/application.scss */
.webpacker-test {
  color: blue;
}
```

然后 `app/javascript/packs/application.js` 最下面添加一行引入 css 文件的声明。

```javascript
require.context('../stylesheets/', true, /^\.\/[^_].*\.(css|scss|sass)$/i)
```

这里第2个参数 `true` 的意思是：搜索也包含全部子文件夹。这样你就可以在 `app/javascript/stylesheets/` 按照自己所需添加子文件夹了。

下面你可以在 view 中添加一点代码，方便一会验证自己的代码生效。

```html
<div style="display:none;">
  <div class="webpacker-test">
    I will be blue if webpacker's CSS is working.
  </div>

  <div class="bootstrap-test text-success">
    I will be green if bootstrap is working.
  </div>

 <div class="jquery-test">
   jQuery is not yet working.
 </div>
 <script>
   $(function(){
     $(".jquery-test").html("jQuery is working.");
   });
 </script>
</div>
```

### 添加 jQuery 和 Bootstrap

因为 Bootstrap 依赖 `jQuery` 和 `popper.js` 所以得同时加上这两个。

```bash
yarn add jquery popper.js bootstrap@4.0.0-beta.2
cp node_modules/bootstrap/scss/_variables.scss app/javascript/stylesheets/
ruby -pi -e 'gsub(/ !default/, "")' app/javascript/stylesheets/_variables.scss
```

Bootstrap 的声明变量的样式复制到我们自定义的文件夹下，然后利用 Ruby 干掉其中的 ` !default` 字符串。

接下来在 `app/javascript/stylesheets/application.scss` 的顶部添加如下三行：

```css
@import '~bootstrap/scss/functions';
@import 'variables';
@import '~bootstrap/scss/bootstrap';
```

然后在 `app/javascript/packs/application.js` 中 `import` jQuery 和 bootstrap。

```javascript
import $ from 'jquery'
global.$ = $
global.jQuery = $

import 'bootstrap'

require.context('../stylesheets/', true, /^\.\/[^_].*\.(css|scss|sass)$/i)
```

最后为了能让 jQuery 和 popper.js 可以在我们写的其它的 js 文件中使用，需要修改 `config/webpack/environment.js`:

```javascript
const { environment } = require('@rails/webpacker')
const webpack = require('webpack')

environment.plugins.set(
   'Provide',
   new webpack.ProvidePlugin({
       $: 'jquery',
       jQuery: 'jquery',
       'window.jQuery': 'jquery',
      Popper: ['popper.js', 'default'],
  })
)

module.exports = environment
```

### 添加 rails-ujs 和 turbolinks

Rails 5.1 鼓励使用 remote/ajax 表单，所以我们 `rails-ujs` 也得用。

```bash
yarn add rails-ujs turbolinks
```

```javascript
// app/javascript/packs/application.js
import Rails from 'rails-ujs';
Rails.start();

import Turbolinks from 'turbolinks';
Turbolinks.start();
```

### Images

Images 和 stylesheets 差不多，假设我们的图片在 `app/javascript/images/` 中，那就在 `app/javascript/packs/application.js` 添加如下：

```javascript
require.context('../images/', true, /\.(gif|jpg|png|svg)$/i)
```



## How to include JS from gems

### 如何通过 Webpacker 的方式集成 `action_cable` 功能？

现在 Rails 团队将其提交到了 https://yarnpkg.com/en/package/actioncable，以后要经常关注 [Yarn Packages]( https://yarnpkg.com/en/packages) 在这里找项目需要用到的类库。

### 如果我想要的 gem 还没在 Yarn Packages 中咋办？

以 cocoon 为例（yarn package 有个 cocoon-rails 也可一试）。

1. 把 gem 解压到 `vendor/gems/cocoon-1.2.10` 并修改 Gemfile，不会的可以参考 [Freeze (vendor, unpack) a single Ruby gem with and without Bundler](https://makandracards.com/makandra/538-freeze-vendor-unpack-a-single-ruby-gem-with-and-without-bundler)。

2. 在 `config/webpacker.yml` 中添加 `vendor` 目录 `resolved_paths: ['vendor']`。

3. 创建 `vendor/gems/index.js` 添加如下代码 `import {} from './cocoon-1.2.10/app/assets/javascripts/cocoon.js';`

4. 在 `app/javascript/application.js` 中 `import {} from 'gems/index.js'; //in /vendor/gems`

来自 [issues#57](https://github.com/rails/webpacker/issues/57)


## 配合 Mina 的部署

最好在部署之前，先去服务器安装好 node 和 yarn 环境，或者也可以直接修改 mina 的部署脚本。

```ruby
# ...
task :setup do
  command %{rbenv install 2.4.2 --skip-existing}
  command %{npm install -g yarn}
end
# ...

desc "Deploys the current version to the server."
task :deploy do
  # uncomment this line to make sure you pushed your local branch to the remote origin
  # invoke :'git:ensure_pushed'
  deploy do
    # Put things that will set up an empty directory into a fully set-up
    # instance of your project.
    invoke :'git:clone'
    invoke :'deploy:link_shared_paths'
    invoke :'bundle:install'
    invoke :'rails:db_migrate'
    command %{yarn install} # 重点是添加这一行
    # command %{NODE_ENV=production bundle exec rails webpacker:compile}
    invoke :'rails:assets_precompile'
    invoke :'deploy:cleanup'

    on :launch do
      invoke :'puma:restart'
    end
  end

  # you can use `run :local` to run tasks on local machine before of after the deploy scripts
  # run(:local){ say 'done' }
end
```

编译的话，可以直接用 mina 的 `rails:assets_precompile`。因为 Webpacker hooks up a new `webpacker:compile` task to `assets:precompile`, which gets run whenever you run `assets:precompile`。如果你不用 Sprockets 的话，就直接把 `webpacker:compile` 添加一个别名叫 `assets:precompile`。

## 相关链接

* [Replacing the Rails 5.1 asset pipeline with webpacker 3](https://iprog.com/posting/2017/11/replacing-rails-51-asset-pipeline-with-webpacker-3)
