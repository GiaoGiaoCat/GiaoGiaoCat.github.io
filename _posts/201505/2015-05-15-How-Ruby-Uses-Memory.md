---
layout: post
title:  "How Ruby Uses Memory"
date:   2015-05-15 11:00:00
categories: ruby
tags: advanced
author: "Victor"
---

每个开发者都想让自己编写的代码占用更少的内存并且运行的更快。在 Ruby 中内存是非常重要的，但是很少有开发者清楚地了解为啥在自己代码运行期间内存占用率会忽高忽低。本文会让你对 Ruby 中内存和对象的关系有一个初步的了解，介绍一些常见的技巧让你的代码减少内存占用并因此运行的更快。

## Object Retention

在 Ruby 中最常见的引起内存飙高的方法是保留对象。Ruby 中的常量是永远不会被垃圾回收的，所以如果常量引用了一个对象，那么这个对象也永远不会被垃圾回收。

```ruby
RETAINED = []
100_000.times do
  RETAINED << "a string"
end
```

我们执行这段代码，并用 `GC.stat(:total_freed_objects)` 观察有多少个对象被释放。让我们对比一下：

```ruby
# Ruby 2.2.0

GC.start
before = GC.stat(:total_freed_objects)

RETAINED = []
100_000.times do
  RETAINED << "a string"
end

GC.start
after = GC.stat(:total_freed_objects)
puts "Objects Freed: #{after - before}"

# => "Objects Freed: 44
````

我们创建了 100000 个 `a string` 的副本，但是由于我们将来 **可能** 会使用它们，所以它们不会被垃圾回收。**在 Ruby 中一个对象一旦被全局对象引用，它就不会被垃圾回收。** 这一原则也适用于常量，全局变量，模块(modules)和类(class)。因此，在全局可访问的任何地方引用对象都要注意这一点。

但是假如我们在这个过程中不保留任何对象：

```ruby
100_000.times do
  foo = "a string"
end
```

被释放的对象会立刻增加到：`Objects Freed: 100038`，内存占用率下降了。当保留对象引用的时候，内存占用从 6mb 增加到 12mb。你也可以使用 `get_process_mem` gem 来监测内存变化。

对象保留也可以使用 `GC.stat(:total_allocated_objects)` 观测，被保留的对象等于 `total_allocated_objects - total_freed_objects`。

## Retention for Speed

Ruby 程序员都很熟悉 **DRY**。这一原则也适用于代码中进行对象分配。有时我们期望保留对象以便重用，而不是一次又一次重新创建。 Ruby 的字符串对象内置了这个方法。冻结一个字符串，解释器会认为你不会修改该字符串，并保留它以便重复使用。下面是一个例子：

```ruby
RETAINED = []
100_000.times do
  RETAINED << "a string".freeze
end
```

执行这个代码，你发现被释放的对象是 `Objects Freed: 50`，看起来没啥变化，但是我们的内存使用率确实降低了。你可以使用 `GC.stat(:total_allocated_objects)` 来验证，我们为 `a string` 分配了很少的对象，因为我们保留并重用了它。

Ruby 只存储一个字符串并引用了 100000 次该对象，而不是创建 100000 个不同的对象。除了降低内存使用，我们还因此减少了运行时间，因为 Ruby 不会浪费时间去创建对象和分配内存。你可以使用 [benchmark-ips](https://github.com/evanphx/benchmark-ips) 来检查。

这个去除重复对象的小技巧虽然常被用来处理字符串,但是当你要把其他对象分配给常量的时候也可以使用。事实上，储存外部连接（例如 Redis）的时候，这个技巧已经成了一种通用模式了。例如：

```ruby
RETAINED_REDIS_CONNECTION = Redis.new
```

因为常量引用了 Redis 的连接，所以它不会被垃圾回收。

很有趣吧，有时我们很小心地保留住对象是可以降低内存占用的。

## Short Lived Objects

大多数对象的生命周期都很短。短的意思是创建对象之后并没有引用它。例如下面的代码：

```ruby
User.where(name: "schneems").first
```

表面上看起来，这个语句仅需要很少的对象（一个hash `{name: "schneems"}`）。事实上，当你调用它的时候，会创建非常非常多的中间对象以便生成正确的 SQL 语句。这些对象中绝大部分的生命周期仅在这段代码的执行过程中。那么，我们为啥要关心这些不会被保留的对象被创建多少个呢？

产生大量生命周期适中和较长的对象会引起内存在一段时间内持续增长。一旦在 GC 释放的瞬间这些对象仍在引用，可能引起 Ruby GC 需要更多的内存。

## Ruby Memory Goes Up

当你有很多对象需要被使用，并且它们超过了 Ruby 当前内存中可放入对象的数量时，Ruby 需要分配更多的内存。从操作系统中请求内存分配的操作是很昂贵的，所以 Ruby 尽量减少这种操作的机会。Ruby 不会每次请求几 KB 的内存，而是请求一大块远超过当前需要的内存。你可以通过设置 `RUBY_GC_HEAP_GROWTH_FACTOR` 环境变量来更改这个值。

例如：Ruby 消耗了 100mb 内存，我们设置 `RUBY_GC_HEAP_GROWTH_FACTOR=1.1`。Ruby 再次请求内存分配的时候，它会得到 110mb 内存。当 Ruby 应用程序启动的时候，它会按照同样的百分比增加内存，直到整个程序可以在这些已分配的内存中执行。这个环境变量值设置越低，意味着我们越要频繁的运行 GC 和请求分配内存。该数值越大，意味着更少的 GC，以及超过我们程序运行所需要的内存。

基于优化网站性能的缘故，很多开发者以为 **Ruby 永远不进行内存释放**。这不完全正确，事实上 Ruby 是会释放内存的，稍后我们会讨论这一点。

如果把这些行为考虑在内，那么你会对于非保留的对象(临时对象)如何影响内存的使用有更清晰的认识。例如：

```ruby
def make_an_array
  array = []
  10_000_000.times do
    array <<  "a string"
  end
  return nil
end
```

当我们调用这个方法，会创建 10000000 个字符串对象。当方法执行完毕退出后，这些字符串没有被引用，所以会垃圾回收。然后，当程序执行期间 Ruby 需要为这 10000000 个字符串分配足够的空间。这大概需要 500mb 的内存。

也许你的应用仅需要 10mb 的空间，但是这个数组的创建却需要分配 500mb 的内存。一个简单的例子，假设这个过程是在一个大型 Rails 项目的页面请求中发生，它会耗尽你的内存。因为如果服务器没有足够的内存，GC 就需要不停地释放和分配内存。

因为分配内存的操作开销很大，Ruby 会把这些分配的内存保持住一段时间。一旦进程将这些内存用尽，那么就再次申请内存。内存会逐渐释放，这一过程很慢。如果你在乎程序的效率，那就尽可能少的创建对象。

## In-Place Modification for Speed

有一个小技巧可以加快程序执行速度和减少对象分配：**利用修改状态来替代创建新对象**。例如，这里有一些代码来自于 [mime-types](https://github.com/halostatue/mime-types) gem:

```ruby
matchdata.captures.map { |e|
  e.downcase.gsub(%r{[Xx]-}o, '')
end
```

这段代码通过正则的 `match` 方法返回了 [matchdata object](http://ruby-doc.org/core-2.2.1/MatchData.html)。然后，它将正则表达式捕获的元素组成了一个数组，并将其传递给代码块。代码块对字符串进行一些处理。这段代码看起来很合理。但是当它在 mime-types gem 中被上千次的调用时，每次调用 `downcase` 和 `gsub` 都会创建一个新字符串对象，及其耗时和浪费内存。为了避免这样，我们可以做一些修改：

```ruby
matchdata.captures.map { |e|
  e.downcase!
  e.gsub!(%r{[Xx]-}o, ''.freeze)
  e
}
```

虽然代码变得冗余一些，但是它运行起来更快。这招可以提高我们的效率，因为我们没有在代码块中引用原字符串对象，所以我们可以放心的修改已经存在的字符串而不是创建一个新的。

注意：你不需要用一个常量来储存正则表达式，所有的正则表达式文本由 Ruby 解释器自动冻结（frozen）。

**In-Place Modification** 也会给你带来麻烦。你很容易修改一个在其它地方会用到的变量，而你并没意识到这一点，因此造成的 bug 很难被找到。在使用这招进行性能优化之前，确保你已经有足够的测试。另外，仅对你仔细斟酌过并确认存在大量创建对象操作的代码进行优化。

有一种错误的观点，认为 **对象是很慢的**。事实上对象可以让程序容易理解和容易优化。即便是最快的工具和技术，当用法不对的时候一样会变得很慢。

在应用级别捕捉不必要的分配可以使用 [derailed_benchmarks](https://github.com/schneems/derailed_benchmarks)。在更底层的级别，可以使用 [allocation_tracer](https://github.com/ko1/allocation_tracer) 或 [memory_profiler](https://github.com/SamSaffron/memory_profiler)。

另外：本文的作者写了 [derailed_benchmarks](https://github.com/schneems/derailed_benchmarks)，可以利用 `rake perf:mem` 来查看内存统计。

## Good to be Free

正如前文所说，Ruby 会释放内存，虽然很慢。执行 `make_an_array` 方法会引起内存飙高，你可以监控 Ruby 是如何释放内存的：

```ruby
while true
  GC.start
end
```

应用程序占用的内存减小的过程非常缓慢。当分配太多内存的时候，Ruby 释放少量空页（一组内存颗粒）。操作系统调用 `malloc` 来进行内存分配，取决于操作系统对于 `malloc` 库的不同实现，这些内存可能会被交还给系统。

对于大多数应用，比如 web 应用来说，这一分配内存的动作都由客户端触发。当客户端频频触发这一动作时，我们无法依靠 Ruby 自身的能力去释放内存来保证我们的应用程序占用的空间足够小。另外，释放内存很耗时，最好还是避免创建对象。

## You’re Up

现在你已经对 Ruby 中对象和内存的关系有了基本的了解。当你想要对自己的程序进行内存方面的性能优化时，可以使用下面的工具：

* [derailed_benchmarks](https://github.com/schneems/derailed_benchmarks)
* [allocation_tracer](https://github.com/ko1/allocation_tracer)
* [memory_profiler](https://github.com/SamSaffron/memory_profiler)
* [benchmark-ips](https://github.com/evanphx/benchmark-ips)

遵循下面的模式：找到引发问题的地方，优化性能，进行性能测试。

## 补充

**Ruby 不会自动释放，只会回收，所以内存消耗只会越来越大。** 参见 [Ruby 的内存陷阱](https://ruby-china.org/topics/25584)。

### GC参数调整

[Ruby 2.1: RGenGC](http://tmm1.net/ruby21-rgengc/) 非常详细地介绍了各个参数的意义，还提供了github用的参数配置。

```ruby
export RUBY_GC_HEAP_INIT_SLOTS=500000
export RUBY_GC_HEAP_FREE_SLOTS=700000
export RUBY_GC_HEAP_GROWTH_FACTOR=1.25
export RUBY_GC_HEAP_GROWTH_MAX_SLOTS=300000
export RUBY_GC_MALLOC_LIMIT=80000000
export RUBY_GC_OLDMALLOC_LIMIT=80000000
```

## Ruby 2.X 之后的改进

### Ruby 2.2 的 可回收 symbol

> Q: 请描述一下 Symbol 可能引起的内存泄露？
> A: Symbol 不会被 GC 回收，如果频繁调用 `#to_sym` 方法将字符串转换成 Symbol 的话，会耗费大量内存。

简单的说，现在这个问题不存在了。`String#to_sym` 和 `String#intern` 获得动态 symbol 都是可以被回收的。参见 [Ruby 2.2 的 可回收 symbol](https://ruby-china.org/topics/21498)。

## 相关链接

* [原文：How Ruby Uses Memory](http://www.sitepoint.com/ruby-uses-memory/)
* [引起内存泄漏的 gems 列表](https://github.com/ASoftCo/leaky-gems)
* [Incremental Garbage Collection in Ruby 2.2](https://blog.heroku.com/incremental-gc)
* [Debugging a Memory Leak on Heroku](https://blog.codeship.com/debugging-a-memory-leak-on-heroku/)
* [The Definitive Guide to Ruby Heap Dumps, Part I](https://blog.codeship.com/the-definitive-guide-to-ruby-heap-dumps-part-i/)
* [The Definitive Guide to Ruby Heap Dumps, Part II](https://blog.codeship.com/the-definitive-guide-to-ruby-heap-dumps-part-ii/)
