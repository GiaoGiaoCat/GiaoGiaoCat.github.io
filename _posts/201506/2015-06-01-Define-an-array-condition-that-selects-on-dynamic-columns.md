---
layout: post
title:  "Define an array condition that selects on dynamic columns"
date:   2015-06-01 23:00:00
categories: rails
tags: tip
author: "Victor"
---

For [some reason](http://makandra.com/notes/740-know-the-side-effects-of-using-hashes-for-conditions) you want to define a find condition in array form. And in that condition both column name and value are coming from user input and need to be sanitized.

Unfortunately this works in SQLite but does not in MySQL:

```ruby
scope :filter, ->(attribute, value) { where([ 'articles.? = ?', attribute, value ]) }
```

The solution is to use sanitize_sql_array like this:

```ruby
scope :filter, ->(attribute, value) {
  where(sanitize_sql_array([ "`articles`.`%s` = '%s'", attribute, value ]))
}
```


### 转自

[Define an array condition that selects on dynamic columns](http://makandracards.com/makandra/741-define-an-array-condition-that-selects-on-dynamic-columns)
