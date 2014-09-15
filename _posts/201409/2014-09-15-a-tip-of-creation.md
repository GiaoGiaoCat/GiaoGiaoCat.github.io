---
layout: post
title:  "%Q, %q, %W, %w, %x, %I, %i, %r, %s"
date:   2014-09-15 14:20:00
categories: ruby
tags: tip
author: "Victor"
---

%W, %Q 相当于双引号，%w, %q 相当于单引号。

### %Q

This is an alternative for double-quoted strings, when you have more quote characters in a string.Instead of putting backslashes in front of them, you can easily write:

```ruby
>> %Q(Joe said: "Frank said: "#{what_frank_said}"")
=> "Joe said: "Frank said: "Hello!"""
```

The parenthesis “```(```…```)```” can be replaced with any other non-alphanumeric characters and non-printing characters (pairs), so the following commands are equivalent:

```ruby
>> %Q!Joe said: "Frank said: "#{what_frank_said}""!
>> %Q[Joe said: "Frank said: "#{what_frank_said}""]
>> %Q+Joe said: "Frank said: "#{what_frank_said}""+
```

You can use also:

```ruby
>> %/Joe said: "Frank said: "#{what_frank_said}""/
=> "Joe said: "Frank said: "Hello!"""
```

### %q

Used for single-quoted strings.The syntax is similar to %Q, but single-quoted strings are not subject to expression substitution or escape sequences.

```ruby
>> %q(Joe said: 'Frank said: '#{what_frank_said} ' ')
=> "Joe said: 'Frank said: '\#{what_frank_said} ' '"
```

### %W

Used for double-quoted array elements.The syntax is similar to %Q

```ruby
>> %W(#{foo} Bar Bar\ with\ space)
=> ["Foo", "Bar", "Bar with space"]
```

### %w

Used for single-quoted array elements.The syntax is similar to %Q, but single-quoted elements are not subject to expression substitution or escape sequences.

```ruby
>> %w(#{foo} Bar Bar\ with\ space)
=> ["\#{foo}", "Bar", "Bar with space"]
```

### %I and %i

%i{ .. } returns a list of symbols without interpolation, %I{ .. } returns a list of symbols with interpolation.

```ruby
>> %i(address city state postal country)
=> [:address, :city, :state, :postal, :country]
```

### %x

Uses the ``` ` ``` method and returns the standard output of running the command in a subshell.The syntax is similar to %Q.

```ruby
>> %x(echo foo:#{foo})
=> "foo:Foo\n"
```

### %r

Used for regular expressions.The syntax is similar to %Q.

```ruby
>> %r(/home/#{foo})
=> "/\\/home\\/Foo/"
```

### %s

Used for symbols.It’s not subject to expression substitution or escape sequences.

```ruby
>> %s(foo)
=> :foo
>> %s(foo bar)
=> :"foo bar"
>> %s(#{foo} bar)
=> :"\#{foo} bar"
```


### 相关链接

* [added symbols and qsymbols productions for %i and %I](https://github.com/ruby/ruby/commit/91bd6e711db3418baa287e936d4b0fac99927711)
