---
layout: post
title:  "Rails 4 flash messages using Twitter Bootstrap"
date:   2015-07-25 23:10:00
categories: rails
tags: tip
author: "Victor"
---

## 方法 1

```erb
<!-- _flash_messages.html.erb -->

<% flash.each do |type, message| %>
  <div class="alert <%= bootstrap_class_for(type) %> fade in">
    <button class="close" data-dismiss="alert">×</button>
    <%= message %>
  </div>
<% end %>
```

```erb
<!-- application.html.erb -->
<%= render "shared/flash_messages", flash: flash %>
```

```ruby
# application_helper.rb
module ApplicationHelper

  def bootstrap_class_for(flash_type)
    case flash_type.to_sym
      when :success
        "alert-success"
      when :error
        "alert-error"
      when :alert
        "alert-block"
      when :notice
        "alert-info"
      else
        flash_type.to_s
    end
  end

end
```

## 方法 2

```erb
<!-- application.html.erb -->
<%= flash_messages %>
```

```ruby
# application_helper.rb

module ApplicationHelper

  def bootstrap_class_for flash_type
    { success: "alert-success", error: "alert-danger", alert: "alert-warning", notice: "alert-info" }[flash_type.to_sym] || flash_type.to_s
  end

  def flash_messages(opts = {})
    flash.each do |msg_type, message|
      concat(content_tag(:div, message, class: "alert #{bootstrap_class_for(msg_type)} alert-dismissible", role: 'alert') do
        concat(content_tag(:button, class: 'close', data: { dismiss: 'alert' }) do
          concat content_tag(:span, '&times;'.html_safe, 'aria-hidden' => true)
          concat content_tag(:span, 'Close', class: 'sr-only')
        end)
        concat message
      end)
    end
    nil
  end
end
```

## 参考

* https://gist.github.com/suryart/7418454
* https://gist.github.com/roberto/3344628
