---
layout: post
title:  "Auto-loading lib files in Rails 4"
date:   2014-10-16 20:10:00
categories: rails
tags: tip
author: "Victor"
---

## If you add a dir directly under app/

啥也不用干。这个目录下的所有文件在产品环境会 eager loaded，在开发环境会 lazy loaded。

## If you add a dir under app/something/

比如 ```app/models/concerns/```, ```app/models/products/```

这取决于你的文件是否在正确的命名空间下。比如 ```app/models/products/``` 内的文件，是否在 ```app/models/products/``` 内。

* yes。啥也不用干。
* no。需要在 ```application.rb``` 中手动引入该文件，方法如下：

```ruby
config.autoload_paths += %W( #{config.root}/app/models/products )
```

最后，这些文件在产品环境下都会 eager loaded。

## If you add code in your lib/ directory

如果在 lib 文件夹进行 **monkey patch** 或者打开一个类，那么不要使用 **autoload** 方法。

* ```config.autoload_paths``` 方法: 如果你的类只在这里定义，那么使用这种方法是有效的。如果类已经在其它的地方定义过，用这种方法你不能重新加载它。
* ```config/initializer/load_rb_file.rb``` 方法: 论你要定义一个新的类，还是打开一个类，或者做一个为已经存在的类添加 **monkey patch**。这个方法都有效。


### 想打开一个类，添加猴子补丁

1. 在 **lib/** 下面添加了一些我想要的文件。
2. 新建一个 ```config/initializers/require_files_in_lib.rb``` 包含如下内容：

```ruby
Dir[Rails.root + 'lib/**/*.rb'].each do |file|
  require file
end
```

### 只是想添加一个新类的时候

首先在 ```config/application.rb``` 文件中添加如下代码

```ruby
config.autoload_paths << Rails.root.join('lib')
```

然后编辑你的文件

```ruby
# lib/foo.rb:
class Foo
end

# lib/foo/bar.rb
class Foo::Bar
end
```

## 相关链接

* [Rails autoloading — how it works, and when it doesn't](http://urbanautomaton.com/blog/2013/08/27/rails-autoloading-hell/#fn1)
* [Rails 3 Autoload Modules/Classes – 3 Easy Fixes](http://www.williambharding.com/blog/technology/rails-3-autoload-modules-and-classes-in-production/)
* [Tips on Rails 3 load paths](http://hakunin.com/rails3-load-paths)
