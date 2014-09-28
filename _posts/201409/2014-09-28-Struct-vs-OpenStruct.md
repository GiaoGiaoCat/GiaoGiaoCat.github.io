---
layout: post
title:  "Struct vs OpenStruct"
date:   2014-09-28 09:40:00
categories: ruby
tags: tip
author: "Victor"
---

## 什么是 Struct?

在 C 语言中, 有一种数据类型叫做 Struct, 定义一个 Struct 相当于定义了一个自定义的数据类型, 里面可以包含各种不同的其它数据类型. 例如:

```c
struct person
{
  int age;
  char name[50];
  char sex;
};
```

同样在 Ruby 中, 也有 Struct, 纯 C 实现, 用于快速声明一个 Class, 例如:

```ruby
Person = Struct.new(:age, :name, :sex)
me = Person.new(24, 'Spirit', 'male')
me.age  # => 24
me.name = 'Test'
me.name # => 'Test'
me.height # => NoMethodError
```

当然, 我们也可以通过定义 Class 来实现同样的功能, 但是 Struct 的快速声明功能是直接定义 Class 无法实现的。

## OpenStruct 详解

在上面的实例中, 可以看到, me 无法动态响应任意的属性, OpenStruct 类恰巧就完成了这件事情.

还是上面那个例子:

```ruby
require 'ostruct'

me = OpenStruct.new(age: 24, name: 'Spirit', sex: 'male')
me.height # => nil
me.height = '178'
me.height # => '178'
```

翻看 **OpenStruct** 的源码, 可以看到其核心为 ```#initialize(), #new_ostruct_member(), #method_missing()``` 三个方法:

```ruby
def initialize(hash=nil)
  @table = {}
  if hash
    hash.each_pair do |k, v|
      k = k.to_sym
      @table[k] = v
      new_ostruct_member(k)
    end
  end
end
```

初始化, 一旦传入的参数为 hash, 则把 hash 的键值对一一保存到实例变量 **@table** 中, 并且调用 ```#new_ostruct_member()``` 方法。

```ruby
def new_ostruct_member(name)
  name = name.to_sym
  unless respond_to?(name)
    define_singleton_method(name) { @table[name] }
    define_singleton_method("#{name}=") { |x| modifiable[name] = x }
  end
  name
end
```

为每一个传入的 name(hash 的 key) 都生成两个实例方法 ```#name``` 与 ```#name=```, 并且定义其值为实例变量 **@table** 中对应的 key 的值

```ruby
def method_missing(mid, *args)
  mname = mid.id2name
  len = args.length
  if mname.chomp!('=')
    if len != 1
      raise ArgumentError, "wrong number of arguments (#{len} for 1)", caller(1)
    end
    modifiable[new_ostruct_member(mname)] = args[0]
  elsif len == 0
    @table[mid]
  else
    err = NoMethodError.new "undefined method `#{mid}' for #{self}", mid, args
    err.set_backtrace caller(1)
    raise err
  end
end
```

重写 ```#method_missing``` 方法

当初始化的时候没有传入 hash, 或者让对象响应一个任意的方法时:

如果需要响应方法为 ```mname=```, 并且实参为一个, 那么调用 ```#new_ostruct_member()``` 方法, 为实例定义两个方法 ```#mname``` 与 ```#mname=```, 下次调用该方法时, 则不需要经过 ```method_missing``` 这一步骤

如果需要响应的方法为 ```mname```, 并且没有实参, 那么直接返回 **@table** 中对应的 k 的值

事实上, 除非为 **@table** 注入一个 ```{ mname: value }``` 的键值对或者调用 ```obj.mname=value```, 否则直接调用 ```obj.mname``` 经过 ```method_missing``` 后返回的值, 都为 **nil**


```ruby
me = OpenStruct.new(age: 24, name: 'Spirit', sex: 'male')
me.height # => 'nil'
me.instance_eval do
  @table[:height] = 178
end
me.height # => '178'
```

ok, ```OpenStruct``` 详解完毕, 其余方法可自行挖掘。

最后说一句, OpenStruct 由于进行了 ```method_missing``` 判断, 其性能与 ```Struct``` 相比, 可谓是天差地别。

### 相关链接

* [原文出处](https://ruby-china.org/topics/21617)
* [Ruby: Struct vs OpenStruct](http://stackoverflow.com/questions/1177594/ruby-struct-vs-openstruct)
