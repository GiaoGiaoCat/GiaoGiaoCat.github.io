---
layout: post
title:  "reverse_merge 为方法设置默认参数的技巧"
date:   2017-05-12 11:30:00
categories: rails
tags: tip
author: "Victor"
---

先来看一下 hash 的 `merge` 方法。

```ruby
h1 = { "a" => 100, "b" => 200 }
h2 = { "b" => 254, "c" => 300 }
h1.merge(h2)   #=> {"a"=>100, "b"=>254, "c"=>300}
h1             #=> {"a"=>100, "b"=>200}
```

Rails 提供的 `reverse_merge` 的作用与 `merge` 正好相反，`merge` 时后面的优先级高，`reverse_merge` 时前面 hash 的优先级高。

`reverse_mergee` 最常见的用处就是在 Rails 中为方法的 hash 参数设置默认值。

```erb
<!-- erb中 -->
<%= display_product @product, :show_price => true %>
```

```ruby
# helper中
def display_product(product, locals = {})
  locals.reverse_merge! :show_price => false
  render :partial => product, :locals => locals
end
```

当然现在看来这个例子并不算很好，因为 [Ruby 2 Keyword Arguments](/ruby/ruby-2-keyword-arguments/)。
