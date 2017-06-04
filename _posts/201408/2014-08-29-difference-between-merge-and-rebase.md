---
layout: post
title:  "git merge 和 rebase 和 cherry-pick 的区别"
date:   2014-08-29 12:05:00
categories: tool
tags: git
author: "Victor"
---

## merge

git merge是用来合并两个分支的。

```bash
# 将b分支合并到当前分支
git merge b
```

## cherry-pick

可以选择某一个分支中的一个或几个commit(s)来进行操作。

它解决的问题就是：在本地 master 分支上做了一个commit ( 38361a68138140827b31b72f8bbfd88b3705d77a ) ，如何把它放到 本地开发 old_cc 分支上？

例如，假设我们有个稳定版本的分支，叫v2.0，另外还有个开发版本的分支v3.0，我们不能直接把两个分支合并，这样会导致稳定版本混乱，但是又想增加一个v3.0中的功能到v2.0中，这里就可以使用cherry-pick了。

```bash
# 先在v3.0中查看要合并的commit的commit id
git log
# 假设是 commit f79b0b1ffe445cab6e531260743fa4e08fb4048b

# 切到v2.0中
git check v2.0

# 合并commit
git cherry-pick f79b0b1ffe445cab6e531260743fa4e08fb4048b
```

## rebase

有点类似git merge，但是两者又有不同，merge适合那种比较琐碎的，简单的合并，系统级的合并还是用rebase吧。

打个比方，你有两个抽屉A和B，里面都装了衣服，现在想把B中的衣服放到A中，git merge是那种横冲直撞型的，拿起B就倒入A里面，如果满了（冲突）再一并整理；而git rebase就很持家了，它会一件一件的从B往A中加，会根据一开始放入的时间顺序的来加，如果满了你可以处理这一件，你可以继续加，或者跳过这一件，又或者不加了，把A还原。

```bash
# 合并b
git rebase b

# 处理完冲突继续合并
git rebase --continue

# 跳过
git rebase --skip

# 取消合并
git rebase --abort
```
