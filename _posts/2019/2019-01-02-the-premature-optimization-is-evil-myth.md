---
layout: post
title:  "提早优化是万恶之源"
date:   2019-01-02 12:00:00

categories: other
tags: knowledge
author: "Victor"
---

过早优化时常出现在我们的开发过程中，不知不觉就会为我们带来不必要的负担。时刻感知到这一问题的存在，知道何时该优化、何时不该优化，是一名优秀的软件工程师必备的素养之一。

## 什么是过早优化

可以先看看 [知乎](https://www.zhihu.com/question/24282796/answer/27279410) 相关的讨论。

> 过早指的不是在开发过程的早期，而是在还没弄清楚需求未来的变化的走向的时候。你的优化不仅可能导致你无法很好地实现新的需求，而且你对优化的预期的猜测有可能还是错的，导致实际上你除了把代码变复杂以外什么都没得到。

### 什么情况下会发生过早优化呢

1. 对所要实现的系统没有一个全面的规划
2. 需求本身不够稳定，或对需求的理解不够透彻

没有全面的规划，就容易拘泥于细节。对于需求分析不够清晰，则往往会制造许多南辕北辙的废代码。在时机仍不成熟时，就针对一些细节上动用太多的脑筋、早早地开始优化性能，不仅会大大地增加代码复杂度，而且这些优化在很多情况还会对后面的开发工作增添大量的障碍。

例如常见的缓存策略。如果缓存基本难以命中倒还好，如果应用一直在使用已经过期的缓存，问题就很大了。一旦在项目中使用了没有经过推敲的缓存失效策略，很容易就会导致很多难以预期的结果。

### 避免过早优化的原则

**先抗住，后优化。**

要避免过早优化，只需要注意两点：

1. 逐步完善系统，不追求一步到位；
2. 在考虑优化的同时，需要清楚它所带来的收益和影响范围。

写代码时需要最优先考虑的应该是 **代码的可读性和可维护性才是**，展开说还包括了代码的复用性、抽象层次。别太担心性能。

> What Andy gives, Bill takes away. —— [安迪-比尔定律](https://baike.baidu.com/item/%E5%AE%89%E8%BF%AA%E6%AF%94%E5%B0%94%E5%AE%9A%E7%90%86?fromId=2066996)

## 反向论点

过早优化是万恶之源。也常常被用来成为程序员偷懒的借口，阻碍进步的动力。对很多程序员来说，其能力还远远达不到过早优化的地步。

> We should forget about small efficiencies, say about 97% of the time: **premature optimization is the root of all evil.** —— [唐纳德·克努特](https://baike.baidu.com/item/%E5%94%90%E7%BA%B3%E5%BE%B7%C2%B7%E5%85%8B%E5%8A%AA%E7%89%B9/1436781?fr=aladdin)


## 相关阅读

* [对“提早优化是万恶之源”的批判](https://gywbd.github.io/posts/2016/10/the-premature-optimization-is-evil-myth.html)
* [不可过早优化](https://www.jianshu.com/p/ef946d489e84)
* [那些害人的编码“神谕”](https://yangwenbo.com/articles/words-that-make-code-bad.html)
* [浅谈前端中的过早优化问题](http://jerryzou.com/posts/talk-about-premature-optimization/)
