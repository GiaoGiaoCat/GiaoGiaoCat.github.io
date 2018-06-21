---
layout: post
title:  "Top 20 Things a Programmer Should Know"
date:   2015-02-07 20:30:00
categories: other
tags: refactoring
author: "Victor"
---

根据28法则，我从《97 Things Every Programmer Should Know》中摘出了其中的 20 条。剔除了其中过于明显和琐碎的内容，书中的许多技巧需要根据上下文才能更好的理解。所以有条件的话请尽量阅读原书。

### 1. Apply Functional Programming Principles

引用透明性是一个非常理想的性质。这意味着，不论函数在何时执行，对于同样的输入总输出同样的结果。没有副作用。

### 2. Ask, "What would the user do?"

你不是用户。我们总是假设用户和我们想的一样，事实并不是这样。用一个小时观察用户如何使用你的程序好过用一天来猜他们的想法。

### 3. Your customers do not mean what they say

在你确认了解客户的需求前请尽可能多的和他们讨论。针对同一个需求最好和客户的不同员工进行沟通，因为有可能他们给你的回复是完全相反的。只有在开发前尽可能多的和他们沟通，才能确保自己了解清楚需求。

最好在交谈和采集需求的时候，使用一些可视化的工具（白板或纸笔都行）。把设计思路记录下来，可以确保在开发阶段更容易在正确的道路上前进。

### 4. Start with Why

不要去想客户是不是要这个功能，而是想他为什么要这个功能。找出背后的原因。你会发现可能有更好的办法来解决客户的问题。

### 5. Hard work does not pay off

连续长时间闷头开发反而会降低工作效率。尽可能的查找更智能的解决方案和提高自己的技能，参加一些技术会议来开拓自己的视野，提高工作效率。

### 6. Do lots of deliberate practice

多练。并不是让你重复一遍又一遍写自己烂熟于胸的代码，而是尽量挑战自己能力之外的项目。

### 7. Reinvent the wheel often

重新发明轮子，可以让你了解各个组件和内部运作的原理。大多数开发人员从来不想重新实现核心的软件库，因此他们也不会深入了解这些软件是如何工作的。

### 8. Continuous learning

阅读。订阅邮件列表。动手编码。找到一个导师和相关的工作。了解你使用的框架和库。当你犯错的时候，修正错误并深入研究它，知道为什么会发生这个错误。教学是学习的好方法。参加一些本地的沙龙和活动。加入或启动一个研究小组。参加业内大会或在线观看这些大会的视频。学习一门新的语言。你可以在目前的技术栈中尝试一些新想法。

### 9. Know how to use command line tools

利用 ``grep`` 和 ``sed`` 进行查找和替换，通常会比 IDE 更好使。

``find . -name '*.rb' | see 's/.*\///' | sort | uniq -c | grep -v "^ *1" | sort -r``

### 10. The Unix tools are your friends

### 11. Step back and automate, automate, automate

### 12. Put everything under version control

### 13. Put the mouse down and step away from the keyboard

### 14. Missing opportunities for polymorphism

尽量使用多态，少用 ``if else`` 和 ``case``。

### 15. Prefer domain-specific types to primitive types

多用特定领域类型而不是基本类型。这样的代码更容易测试和重用。

### 16. Test for required behavior, not incidental behavior

仅测试必然的行为，而不是偶然的行为。过于测试某些偶然会发生的行为是没意义的。

### 17. Test precisely and concretely

对于特定的行为，测试不应该仅仅是正确的，它必须精确。将一个对象添加到一个集合中，你要测试的并不是结果不为空。而是应该测试结果结合仅含一个对象，并且就是刚刚我们添加进去的对象。

### 18. The golden rule of API design

仅仅为你设计的 API 编写测试是不够的，必须要为这些 API 调用的方法编写单元测试。

### 19. Write Tests for People

良好的测试就是代码的文档。它描述代码是如何工作的。对于每个使用场景，这些测试：

* 描述上下文，起点和必须满足的前提条件
* 说明软件如何被调用
* 描述了预期的结果或对提交的条件进行验证

不同的场景可能会有细微的不同。但是其他人可以通过查看几个测试就能了解软件的功能，每个测试都应该清楚的说明这三部分。

### 20. One binary rule

Build a single binary that you can identify and promote through all the stages in the release pipeline. Hold environment-specific details in the environment. This could mean, keeping them in a file or in the path. Keep the environment information versioned too. If the environment configuration breaks, without versioning we cannot figure out what changed. The environmental information should be versioned separately from the code, since they’ll change at different rates and for different reasons.
