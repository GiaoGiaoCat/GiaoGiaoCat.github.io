---
layout: post
title:  "You’ll love module_function"
date:   2016-01-10 12:00:00
categories: ruby
tags: tip
author: "Victor"
---

`module_function` 可以把一个方法定义成实例方法同时也是 module level methods。

你可以给 `module_function` 传递一个方法名作为参数：

```ruby
module X
  def foo
    'bar'
  end
  module_function :foo
end

$ X.foo
=> 'bar'
```

如果 `module_function` 没有跟随参数，则它后面的所有方法都会被定位为 module level methods：

```ruby
module X
  module_function
  def foo
    'huhu'
  end
end

$ X.foo
=> 'huhu'
```

所有通过 `module_function` 定义的方法默认都是 `private` 的。

更妙的是，这些 module level methods 是实例方法的副本，所以给其中的一个方法打猴子补丁不会影响另外的。

### Why should I be using this?

下面是一个引用 `Erb::Util` 的例子，通常我们这样使用模块方法：

```ruby
require 'erb'
class X
  include ERB::Util
  def call
    url_encode 'http://www.google.com'
  end
end

$ X.new.call
=> 'http%3A%2F%2Fwww.google.com'
```

或者也可以这样

```ruby
require 'erb'
class Y
  def call
    ERB::Util.url_encode 'http://www.google.com'
  end
end

$ Y.new.call
=> 'http%3A%2F%2Fwww.google.com'
```

因为使用 `include` 混入模块会影响 `is_a?` 类型检查和条件比较符 `===`。

所以利用 `module_function` 我们可以直接使用 `url_encode` 而不需要破坏继承链。

### 原文

* [You’ll love module_function](https://medium.com/raise-coffee/you-ll-love-module-function-ca5f05284310#.yrb1rokzr)
