---
layout: post
title:  "在 Ruby 中检查和更新 URL 的一个小技巧"
date:   2018-11-27 12:00:00

categories: ruby
tags: gems
author: "Victor"
---

需要在一个历史遗留的数据库中清理和更新其中的 URL。这些 URL 各种各样：有些是 http/https、有些可能都没有 http:// 和 www 部分、有些网址已经不能访问了。

我们需要一个解决方案，传给它一个现在数据库的 URL，它返回一个可用的完整 URL，如果网址已经不能访问就返回 `nil`。

说着容易做着难，在各种尝试之后最后选定了基于 libcurl 的 [curb](https://github.com/taf2/curb)。

```ruby
def checked_url(url)
  begin
    result = Curl::Easy.perform(url) do |curl|
      curl.head = true
      curl.follow_location = true
      curl.timeout = 3
    end
    result.last_effective_url
  rescue
    nil
  end
end
```

```bash
irb(main):042:0> checked_url('nyt.com') #=> "https://www.nytimes.com/"
```

## 原文链接

* [Check and Update a URL with Ruby](https://medium.com/@wintermeyer/check-and-update-a-url-with-ruby-120e6ba73e4f)
