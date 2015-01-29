---
layout: post
title:  "Growing Rails Applications in Practice - 6"
date:   2015-01-29 20:30:00
categories: rails
tags: learningnote refactoring
author: "Victor"
---

## Building applications to last - 下

### Owning your stack

对于我们想要的功能，在 [Ruby Toolbox](https://www.ruby-toolbox.com/) 中几乎都能找到相关的 gem。听起来我们只要把这些 gems 绑在一起就能快速完成一个应用。然而，你要知道将组建加入我们的项目中所付出的代价。

当你把一个 gem 或技术引入你的技术栈的时候，你要明白 **you now own that component**，这句话意味着：

* 你需要为它引起的行为负责。
* 你要为它将来的安全更新负责。
* 当你更新技术栈的时候有责任对其升级，例如：新版本的 Rails 引起该 gem 的失效。
* 如果该 gem 的维护者不再维护，你需要有能力更换或接手维护它。
* 你也要为这个 gem 的依赖，以及依赖的依赖负同样的责任。往技术栈引入一个 gem 往往意味着引入一打 gems。

**You must own your stack。**

当你无法衡量引入一项新的类库的成本时，你可以：

* 浏览 github 上的代码，查看它的代码是否看起来整齐干净，还是有一个又臭又长的方法和类。
* 它是否有测试？你可以看 github 上的 test 或 spec 目录，确保它包含测试文件而不是自动创建的文件夹。
* 它是否仍在开发阶段？检查 github 上的 commits 和 issues 列表，查看是否有人回复处理这些问题。
* 当作者不维护的时候，你是否做好准备来继续维护它？
* 类库是否提供足够的功能便于整合和将来维护。花一分整理一下如果自己重做同样功能所需要的工作。例如：你可以通过一个新的控制器和几行 jQuery 代码来完成 autocomplete suggestions 功能，而不是引入一个新的类库。但是如果要自己实现 XML 解析功能就要实现一大堆功能以便兼容各种复杂的格式。

对是否引入一个第三方类库到你的技术栈做出明智的决定，能够加快你的开发，而不是破坏日后的维护工作。

#### Accepting storage services into your stack

每周 Hacker News 上都出现新的数据库解决方案趋势，都号称可以解决你管理和扩展数据上的痛苦。当你在技术栈引入意向新的存储技术时，你要承担和引入一项新类库同样的责任。

在你决定引入新的存储技术前，这里有一些建议：

* 它的负载怎么样？
* 它是分布式的吗？有何保证？
* 如何处理硬件故障？
* 是否有成熟的 Ruby 类库？
* 能否不锁的情况下备份数据？
* 能否方便快速的访问到所需类型的数据？
* 有成熟的项目在产品环境中使用它吗？

#### Maxing out your current toolbox

另外一种向项目中增加新技术的方法是，查看当前已有的工具是否足够完成这个任务。你可以延后再把事情变得复杂。

一些例子：

Requirement | New technology | Technology you already have
---|---|---
Key-value store | Redis | Another MySQL table
Full text search | Lucene / Solr / Sunspot | MySQL LIKE queries or FULLTEXT indexes
Background tasks | Sidekiq queue | [Cron job](https://github.com/javan/whenever)

注意，我们并不是建议你拒绝新技术。我们的认为，在引入新技术前你应该用一种更简单的解决方案，直到你的负载和数据容量让你不得不转换。

为了以后可以轻松替换一个新的解决方案，我们建议利用 service objects 并仅暴露简单的 API 而隐藏后面的服务。假设你要实现一个全文检索功能，那么最好是先用 MySQL 的 LIKE 查询而不是集成一个像 Solr 这样的索引服务。

相反，不要在代码中直使用 MySQL 查询，而是利用 service object 来隐藏这些细节：

```ruby
Article::Search.find('query goes here')
```

这样，当你的数据已经超过了 MySQL 的容量，你需要一个更复杂的替代品，你可以很简单的修改 ``Article::Search.find`` 方法而不用碰其它代码。

### The value of tests

#### Choosing test types effectively

#### How many tests are too many?

#### When to repeat yourself in tests - and when not to

#### Better design guided by tests

#### Getting started with tests in legacy applications

#### Overcoming resistance to testing in your team
