---
layout: post
title:  "How to return correct HTTP error codes from Ruby on Rails application"
date:   2015-06-02 17:50:00
categories: rails
tags: tip
author: "Victor"
---

在 Rails 中有两种方法，返回 HTTP error code。

```ruby
head 403
head :forbidden
```

### 相关链接

* [ActionController::Head](http://api.rubyonrails.org/classes/ActionController/Head.html)
* [Rack::Utils::SYMBOL_TO_STATUS_CODE](http://www.rubydoc.info/github/rack/rack/Rack/Utils)
