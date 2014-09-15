---
layout: post
title:  "How to parse String to Hash in Ruby"
date:   2014-09-12 17:10:00
categories: ruby
tags: tip
author: "Victor"
---

### Parsing JSON

To parse a JSON string received by another application or generated within your existing application:

```ruby
require 'json'

my_hash = JSON.parse('{"hello": "goodbye"}')
puts my_hash["hello"] => "goodbye"
```

```ruby
require 'rubygems'
require 'json'

content = %Q({\"credentials\"=>{\"token\"=>\"F538FF45937887AF7246E50928E0961F1\", \"refresh_token\"=>\"6EC400DD7B547B401D29474EA68952145\", \"expires_at\"=>1418310654, \"expires\"=>true}})

hash = JSON.parse content.gsub!('=>', ':')

puts hash.class
#=> Hash
```

Notice the extra quotes '' around the hash notation. Ruby expects the argument to be a string and can’t convert objects like a hash or array.

Ruby converts your string into a hash.

### Generating JSON

```ruby
require 'json'

my_hash = {:hello => "goodbye"}
puts JSON.generate(my_hash) => "{\"hello\":\"goodbye\"}"
```

Or an alternative way:

```ruby
require 'json'
puts {:hello => "goodbye"}.to_json => "{\"hello\":\"goodbye\"}"
```
