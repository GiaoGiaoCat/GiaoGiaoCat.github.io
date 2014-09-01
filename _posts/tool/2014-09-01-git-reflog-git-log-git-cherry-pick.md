---
layout: post
title:  "git reflog 和 git log 的区别，外加 git cherry-pick 的一种用法"
date:   2014-09-01 10:30:00
categories: tool
tags: git
author: "Victor"
---

## git reflog

可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作），

## git log

则不能察看已经删除了的commit记录

具体一个例子，假设有三个commit:

```
git st:

commit3: add test3.c
commit2: add test2.c
commit1: add test1.c
```

如果执行 ```git reset --hard HEAD~1 ``` 则删除了 commit3，如果发现删除错误了，需要恢复 commit3 就要使用

```
git reflog HEAD@{0}: HEAD~1: updating HEAD
63ee781 HEAD@{1}: commit: test3:q
```

63ee781 即是被删除了的 commit3，运行 git log 则没有这一行记录

可以使用 ```git reset --hard 63ee781``` 将红色记录删除，则恢复了cmmit3，运行git log后可以看到：

```
commit3: add test3.c
commit2: add test2.c
commit1: add test1.c
```

这里也可以使用另外一种方法来实现：```git cherry-pick 63ee781```
