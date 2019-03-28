---
layout: post
title:  "Go 的设计模式"
date:   2019-03-27 13:00:00

categories: go
tags: refactoring
author: "Victor"
---

## 设计模式的主要原则

先复习一下自己以前写的 [面向对象设计实践指南 1](http://wjp2013.github.io/ruby/Practical-Object-Oriented-Design-in-Ruby-1/)。

1. 单一职责原则
  * 尝试用一句话来描述类。如果这种描述中出现 “或、和、并” 这样的字，那么这个类就具备了多种职责。
2. 开闭原则
  * 对扩展开放，对修改关闭。
  * 表象上看是在程序需要进行拓展的时候，不能去修改原有的代码。深层上看要使用接口和抽象类。
3. 里氏代换原则
  * 子类可以替代父类。
4. 接口隔离原则
  * 客户端不应该强行依赖它不需要的接口，类间的依赖关系应该建立在最小的接口上。
  * **使用多个隔离的接口，比使用单个接口要好。**
  * Go 默认推荐，因为其推荐单一接口，Ruby 中使用职责更内聚的 module。
5. 依赖倒转原则
  * 对接口编程，依赖于抽象而不依赖于具体。
6. 迪米特法则
  * 最少知道原则，一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。
  * **将某条消息通过第二个不同类型的对象转发给第三个对象。**
7. 合成复用原则 Composite Reuse Principle
  * 原则是尽量使用合成/聚合/组合的方式，而不是使用继承。

### 依赖倒转原则

类 A 直接依赖于类 B，假如要将类 A 修改为依赖类 C，则必须通过修改类 A 的代码来达成。这种场景下，类 A 一般是高层模块，负责复杂的业务逻辑。类 B 和 C 是底层模块，负责基本的原子操作。假如修改类 A，将会给程序带来不必要的风险。

http://www.cnblogs.com/hellojava/archive/2013/03/18/2966684.html

## 相关链接

1. [设计模式 Golang实现－《研磨设计模式》读书笔记](https://github.com/senghoo/golang-design-pattern)
2. [超赞的GO语言设计模式和成例集锦](https://blog.csdn.net/joy0921/article/details/80125194)
3. [Go Patterns](https://books.studygolang.com/go-patterns/)
4. [Some useful patterns](https://blogtitle.github.io/some-useful-patterns/)

按顺序阅读，2 是 3 的中文目录。
