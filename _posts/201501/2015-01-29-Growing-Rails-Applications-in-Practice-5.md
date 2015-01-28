---
layout: post
title:  "Growing Rails Applications in Practice - 5"
date:   2015-01-29 18:30:00
categories: rails
tags: learningnote refactoring
author: "Victor"
---

## Building applications to last

### On following fashions

Ruby on Rails 的生态系统也并非一成不变的。网络上总是一天到晚的为新的设计模式或新 gem 而争吵，他们总号称能彻底改变你对 Ruby 的思考方式。

很难判断哪个框架或技术会流行，而哪个仅仅是昙花一现。

#### Before/after code comparisons

实践新技术的好办法是尝试利用新技术重写应用中的一小部分，并对比重写前后的代码。然后通过对比这些对吗，你要问自己：

* 重构后的代码是否更易读？你能用语言来证明你的想法吗？
* 代码量如何？修改了多少文件？多少类被解耦？如果增加了工作或复杂度，它是否胜过你获得的好处。
* 是否因为这种修改而引入新的问题？

如果有同事，问问他们的意见。

如果新技术需要新的类库，可以考虑升级(下一章会讲 **Surviving the upgrade pace of Rails**)。

一旦你理解了新的设计模式，那么就要权衡一下它是否适合你的工作，或者干脆放弃它。

#### Understanding trade-offs

没有银弹。新的设计模式总在不断的改进，或者被另外的设计模式替代。 一种技术可能有漂亮的语法，但是让你的类更难测试。一种模式让你的代码很短，但是幕后却有无数的黑魔法没准消耗很多性能。

你要学会如何快速了解一种新的模式，并权衡它对你项目的利弊。回头看我们前面的 **note on controller abstractions** 部分，看看我们是如何一步步分析这项技术的优劣并权衡是否该引入我们的项目中的。

#### The value of consistency

我们相信借助 Rails 默认的 MVC 架构，配以良好的组织架构和设计模式会让我们走的更远。

如果总是跟随每一项新技术，你的代码有可能被各种风格代码拼凑而成。这将让项目难以理解和修改。当引入一项新技术，你需要考虑迁移完成应用的成本和团队成员的心理压力。

如果希望应用保持一致的代码风格，这种开销可能是值得的。但你需要的可能不是适应某个高级的技巧而是花时间重构全部代码。

### Surviving the upgrade pace of Rails

和 Java 相反，Rails 圈保持有限的 APIs 向后兼容。因此，新版本的 Rails 和 Ruby gems 经常破坏现有代码，不停的升级非常耗费时间。

不升级当然是一个选择。一旦你落后了太多的版本，那么你无法享受新版本的安全更新。鉴于一些过去被披露的漏洞，你可能不希望在互联网上发布一个没打补丁的 Rails 程序。

有一些产品专门为旧版本的 Rails 提供安全补丁，比如 [Rails LTS](https://railslts.com/)。不过如果你想利用更新的功能，你会发现需要无休止的每几个月就升级应用程序。

本章将对如何处理这种现状提供一些建议。

#### Gems increase the cost of upgrades

当增加一个新的 gem 依赖时，考虑一下在你项目周期中升级这个 gem 的成本。gem 的作者是否在接下来的两年仍然会维护这个项目？如果该 gem 不随 Rails 版本刚更新，那么迫不得已时你能否替换掉这个 gem 或者继续维护它？

请意识到低级抽象类库和高度耦合的 mini 框架之间不同的升级成本。例如：一个支持提供地理位置计算并不依赖于 Rails 的类库，它的升级不会破坏你的系统。另外一个 gem 动态生成管理后台的界面，它肯定要在内部调用 Rails 的方法，那么必然要随着 Rails 一起更新。

#### Upgrades are when you pay for monkey patches

Ruby 强大的元编程能力让你可以在重载类进行 Monkey patches，修改一些 gem 内部依赖的方法或自定义行为。虽然这可能让你快速修复某个类库的 bug，但当你升级 Rails 的时候，基本上 monkey patches 肯定是第一个出问题的。

要小心在一个不属于你的类的内部出现 monkey patching。否则你将为这个情况付出代价。

If you find a bug in a gem that you like, consider forking the gem, committing your fix with a test and creating a pull request to the original author. This way your monkey patch can soon be replaced with the next official version of the gem. It also makes for good karma.

#### Don’t live on the bleeding edge

不需要在第一时间跟随 Rails 新版本升级。直到大版本已经发了一些小布丁之后再升级。例如：不要直接立刻升级到 Rails 5.0.0，等到 5.0.3 发布的时候再升级。

这会给你的 gem 维护者一些时间让代码支持 Rails 新版本。

