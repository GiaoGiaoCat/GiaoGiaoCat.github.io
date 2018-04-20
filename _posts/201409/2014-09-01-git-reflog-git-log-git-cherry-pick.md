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

这里也可以使用另外一种方法来实现：`git cherry-pick 63ee781`

### Git log 常用选项

| 选项 | 说明 |
|---|---|
| -p | 按补丁格式显示每个更新之间的差异 |
| --stat | 显示每次更新的文件修改统计信息 |
| --shortstat | 只显示 --stat 中最后的行数修改添加移除统计 |
| --name-only | 仅在提交信息后显示已修改的文件清单 |
| --name-status | 显示新增、修改、删除的文件清单 |
| --abbrev-commit | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符 |
| --relative-date | 使用较短的相对时间显示（比如，“2 weeks ago”） |
| --graph | 显示 ASCII 图形表示的分支合并历史 |
| --pretty | 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式） |

### 限制输出长度

* 除了用 `-n` 来限制输出 log 的条数，还可以用 `--since` 和 `--until` 按照时间作限制。
* 用 `--author` 选项显示指定作者的提交，用 `--grep` 选项搜索提交说明中的关键字。
* 要得到同时满足这两个选项搜索条件的提交，就必须用 `--all-match` 选项。
* `-S` 列出那些添加或移除了某些字符串的提交。
* 可以在 git log 选项的最后指定它们的路径。因为是放在最后位置上的选项，所以用两个短划线 `--` 隔开之前的选项和后面限定的路径名。

```bash
# 列出所有最近两周内的提交
$ git log --since=2.weeks

# 想找出添加或移除了某一个特定函数的引用的提交
$ git log -Sfunction_name

# 2018 年 4 月期间，Junio Hamano 提交的但未合并的测试文件
$ git log --pretty="%h - %s" --author=gitster --since="2018-04-01" --before="2018-05-01" --no-merges -- t/
```
