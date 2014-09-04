---
layout: post
title:  "Using concat and capture to clean up custom Rails helpers"
date:   2014-09-04 11:40:00
categories: rails
tags: helpers
author: "Victor"
---

You can use the built-in Rails helpers, such as **content_tag** or **link_to**, in your own helpers.

If you need to concatenate them, you could use **+**

```ruby
module MyHelper
  def widget
    content_tag(:p, class: "widget") do
      link_to("Hello", hello_path) + " " + link_to("Bye", goodbye_path)
    end
  end
end
```

But this gets fiddly quick.

### concat

Instead, you can use **concat**

```ruby
module MyHelper
  def widget
    content_tag(:p, class: "widget") do
      concat link_to("Hello", hello_path)
      concat " "
      concat link_to("Bye", goodbye_path)
    end
  end
end
```

Each **concat(content)** adds that content to the output buffer, much like ```<%= content %>``` will in a template file.

### capture

There’s a gotcha when you use **concat** outside a block helper (outside e.g. a **content_tag** or **link_to** block).

Say you have this:

```ruby
module MyHelper
  def widget
    concat link_to("Hello", hello_path)
    concat " "
    concat link_to("Bye", goodbye_path)
  end
end
```

You might expect this to produce no output, but you’ll see the output once:

```<% widget %>```

You might expect this to produce the output once, but it will appear twice:

```<% widget %>```


Why? Inside block helpers, **concat** appends only to that element’s content; outside block helpers, it appends straight to the page itself, and is then appended to the page again as ```<%= widget %>``` writes out the helper’s return value.

You can get around this with the **capture** helper. That’s what **content_tag** uses internally: **capture** , true to its name, captures the content so it isn’t appended straight to the page. Instead, it’s built up as a separate string, for your custom helper to return (or have its way with):

```ruby
module MyHelper
  def widget
    capture do
      concat link_to("Hello", hello_path)
      concat " "
      concat link_to("Bye", goodbye_path)
    end
  end
end
```

If you find yourself writing a lot of complex markup in a helper, you may of course be better served by rendering a partial – perhaps from within your helper. But for those cases with a lot of logic influencing the choice of elements or attributes, where a helper may be the best option, **concat** and **capture** can clean things up a bit.

### 原文来自

* [Using concat and capture to clean up custom Rails helpers](http://thepugautomatic.com/2013/06/helpers/)
* [相关代码](http://apidock.com/rails/ActionView/Helpers/TextHelper/concat)
