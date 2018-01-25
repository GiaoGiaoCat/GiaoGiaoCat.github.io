---
layout: post
title:  "Auto-loading lib files in Rails 5"
date:   2014-10-16 20:10:00
categories: rails
tags: tip
author: "Victor"
---

## Rails autoload

首先我们需要知道 Rails autoload 的工作原理。Rails 提供了一种机制，让用户不需要在应用程序文件中 `require` 的每一个依赖。

如果我们在 Rails 中调用任何没有 load 过的常量，Rails 将尝试在加载路径中查找文件，一旦找到就立刻 `require` 它。

例如，当我们调用 `Product`，Rails 会去 `app/models`，`app/controllers`，`lib/` 和其它加载路径中查找 `product.rb` 文件。Rails 通过扩展 ruby 的 `const_missing?` 方法来实现上述功能。

```ruby
# in active_support/dependencies.rb
def const_missing(const_name)
  from_mod = anonymous? ? guess_for_anonymous(const_name) : self
  Dependencies.load_missing_constant(from_mod, const_name)
end

# Dependencies.load_missing_constant
# lib/active_support/dependencies.rb:477
expanded = File.expand_path(file_path)
expanded.sub!(/\.rb\z/, '')
if loading.include?(expanded)
  raise "Circular dependency detected while autoloading constant #{qualified_name}"
end
```

所以当 Rails `require` 或 `autoload` 文件时，它会记录已经加载过的文件，并在加载相同的文件时引发错误。

### Rails circular dependency Circular dependency

```ruby
# ./app/models/alpha_product
class AlphaProduct < BaseProduct
end
# ./app/models/base_product.rb
class BaseProduct
  PRODUCTS = [AlphaProduct, Product]
  # this works
  # PRODUCTS = [Product]
end
# ./app/models/product.rb
class Product < BaseProduct
end
# test file:
require 'spec_helper'
it 'does something' do
  AlphaProduct.do_things # RuntimeError: Circular dependency detected while autoloading constant AlphaProduct
end
```

上面的代码，当加载 `alpha_product.rb` 时会自动加载 `base_product.rb`，而 `base_product.rb` 中又会去自动加载 `alpha_product` 依赖，故而会引发错误。

但是，当我们尝试首先加载 `base_product` 时，它会创建 `BaseProduct` 类，并自动加载子类。当子类对 `BaseProduct` 的依赖性被调用时，该类已经是被 `require` 过，因此它不会触发自动加载。所以不会引起错误。

## Eager loading

现在我们已经知道 `circular dependency` 错误是怎么发生的，但是为什么只有在运行测试时才会失败？
为了搞懂这一点，我们要了解 **加载顺序** 和 `eager loading` 的问题。

在测试环境，我们会设置 `config.eager_loading = true`，它的意思是预加载所有在 `eager loading paths` 之中的文件。

```ruby
# railties/lib/rails/engine.rb
# Eager load the application by loading all ruby
# files inside eager_load paths.
def eager_load!
  config.eager_load_paths.each do |load_path|
    matcher = /\A#{Regexp.escape(load_path.to_s)}\/(.*)\.rb\Z/
    Dir.glob("#{load_path}/**/*.rb").sort.each do |file|
      require_dependency file.sub(matcher, '\1')
    end
  end
end
```

当 `eager_loading` 是 `true` 的时候：

1. Rails 会将 `eager loading paths` 中的文件排序，并调用 `require_dependency` 方法。
2. `require_dependency` 使用与 `autoload` 相同的 `require_or_load` 方法，故此也会记录加载过的文件。

所以 `alpha_product.rb` 总是会在 `base_product.rb` 之前被加载，也就会引起 `circular dependency` 错误。

### how about Product

`product.rb` 从排序上来说会在 `base_product.rb` 之后加载。当加载 `base_product.rb` 的时候会加载相关依赖。所以当运行 `Product` 相关测试不会引发 `circular dependency` 错误。

### 小结

For alpha product:

1. loading AlphaProduct
2. detected const missing for BaseProduct, before AlphaProduct declare
3. autoload BaseProduct
4. detected const missing for AlphaProduct
5. autoload AlphaProduct
6. detected circular dependency

For product:

1. loading BaseProduct
2. detected const missing for Product, after BaseProduct declare
3. autoload Product with dependency of BaseProduct

## Tips on Rails load paths

### If you add a dir directly under app/

啥也不用干。这个目录下的所有文件在产品环境会 eager loaded，在开发环境会 lazy loaded。

### If you add a dir under app/something/

比如 ```app/models/concerns/```, ```app/models/products/```

这取决于你的文件是否在正确的命名空间下。比如 ```app/models/products/``` 内的文件，是否在 ```module Products``` 内。

* yes。啥也不用干。
* no。需要在 ```application.rb``` 中手动引入该文件，方法如下：

```ruby
config.autoload_paths += %W( #{config.root}/app/models/products )
```

最后，这些文件在产品环境下都会 eager loaded。

### If you add code in your lib/ directory

如果在 lib 文件夹进行 **monkey patch** 或者打开一个类，那么不要使用 **autoload** 方法。

* ```config.autoload_paths``` 方法: 如果你的类只在这里定义，那么使用这种方法是有效的。如果类已经在其它的地方定义过，用这种方法你不能重新加载它。
* ```config/initializer/load_rb_file.rb``` 方法: 论你要定义一个新的类，还是打开一个类，或者做一个为已经存在的类添加 **monkey patch**。这个方法都有效。

## 实际使用

### 想打开一个类，添加猴子补丁

1. 在 **lib/** 下面添加了一些我想要的文件。
2. 新建一个 `config/initializers/require_files_in_lib.rb` 包含如下内容：

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

## Rails 5 补充

因为线程安全的问题，`Autoloading` 在 Rails 5 默认被禁用了。但是只要使用 `config/initializers/require_files_in_lib.rb` 的方式 `require` 在 `lib` 目录下的文件，就不用担心多余的。

## 相关链接

* [Rails autoloading — how it works, and when it doesn't](http://urbanautomaton.com/blog/2013/08/27/rails-autoloading-hell/#fn1)
* [Rails 3 Autoload Modules/Classes – 3 Easy Fixes](http://www.williambharding.com/blog/technology/rails-3-autoload-modules-and-classes-in-production/)
* [Tips on Rails 3 load paths](http://hakunin.com/rails3-load-paths)
* [Load lib files in production (Rails 5)](https://gist.github.com/maxivak/381f1e964923f1d469c8d39da8e2522f)
