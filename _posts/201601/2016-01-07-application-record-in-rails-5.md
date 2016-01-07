---
layout: post
title:  "ApplicationRecord in Rails 5"
date:   2016-01-07 08:00:00
categories: rails
tags: rails5
author: "Victor"
---

Rails 5 beta-1 发布后，引入了一个值得注意的特性 [ApplicationRecord](https://github.com/rails/rails/pull/22567)。

Rails 4.2 之前的版本，所有 models 都继承自 `ActiveRecord::Base`。但是 Rails 5 开始，所有 models 都要改去继承 `ApplicationRecord`。

```ruby
class Post < ApplicationRecord
end
```

你可能会问 `ActiveRecord::Base` 哪去了？

其实没啥变化，Rails 5 生成的项目会自动添加一个新文件。

```ruby
# app/models/application_record.rb
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
```

没错，这看起来就跟 controller 继承 `ApplicationController`，然后由 `ApplicationController` 继承 `ActionController::Base` 一样。

现在 `ApplicationRecord` 是自定义类库和扩展的唯一入口，我们再也不需要给 `ActiveRecord::Base` 做猴子补丁了。

在 Rails 4.2 的时代，如果要给 `Active Record` 增加点功能，我们需要：

```ruby
module MyAwesomeFeature
  def do_something_great
    puts "Doing something complex stuff!!"
  end
end

ActiveRecord::Base.include(MyAwesomeFeature)
```

这样的缺点是显而易见的，`ActiveRecord::Base` 总是会带有 `MyAwesomeFeature` 的功能，所有继承了它的 model 也都有了这些功能，不管它们是否需要这个功能。

当使用 plugins 和 engines 给 `ActiveRecord::Base` 做猴子补丁的时候，这就很容易影响到 plugins 和 engines 之外的代码。造成不同 plugins 和 engines 之间的冲突。

但是借助 `ApplicationRecord`，我们可以将这种影响缩小。

```ruby
class ApplicationRecord < ActiveRecord::Base
  include MyAwesomeFeature

  self.abstract_class = true
end
```

### Migrating from Rails 4

默认情况下，所有 Rails 5 的应用都会自动生成 `application_record.rb` 文件。如果我们的代码是从 Rails 4 迁移过来，只需要自己创建 `app/models/application_record.rb` 文件，并修改所有 models 文件。不再继承 `ActiveRecord::Base` 而改去继承 `ApplicationRecord` 就好。

### 原文

* [ApplicationRecord in Rails 5](http://blog.bigbinary.com/2015/12/28/application-record-in-rails-5.html)
