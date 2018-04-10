---
layout: post
title:  "git 删除错误提交的 commit"
date:   2014-08-29 11:41:00
categories: tool
tags: git
author: "Victor"
---

## 情况1

起因: 不小新把记录了公司服务器IP,账号,密码的文件提交到了git

方法:

    git reset --hard <commit_id>
    git push origin HEAD --force


其他:

    根据–soft –mixed –hard，会对working tree和index和HEAD进行重置:
    git reset –mixed：此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
    git reset –soft：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
    git reset –hard：彻底回退到某个版本，本地的源码也会变为上一个版本的内容


    HEAD 最近一个提交
    HEAD^ 上一次
    <commit_id>  每次commit的SHA1值. 可以用git log 看到,也可以在页面上commit标签页里找到.


## 情况2

又一次: 把 master 的提交路线搞成了迷宫

![](https://raw.githubusercontent.com/wjp2013/the_room_of_exercises/master/assets/wiki/001.jpg)

方法:

1. 到 master 分支利用命令 ```git reset <commit_id>```
2. git push origin HEAD --force

这里的 commit_id 要用分叉之前的，这样所有的代码不会丢，develop 分支上的提交信息也不会丢。那些奇怪的 merge 信息就没了。

## 情况3

一个 feature 有太多琐碎的 commit 想在 push 之间好好清理一下思路，汇合提交：

* 重新输入正确的提交注解
* 清楚地汇合内容含义相同的提交
* 添加最近提交时漏掉的档案

rebase 指定 i 选项，您可以改写、替换、删除或合并提交。

```bash
$ git rebase -i HEAD~~

pick 9a54fd4 添加commit的说明
pick 0d4a808 添加pull的说明

# Rebase 326fc9f..0d4a808 onto d286baa
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

将第二行的 pick 改成 squash，然后保存并退出。由于合并后要提交，所以接着会显示提交信息的编辑器，请编辑信息后保存并退出。

## 情况4

上上次的 commit 内容只写了一个 fix，不符合团队规范要求。

```bash
$ git rebase -i HEAD~~

pick 9a54fd4 添加commit的说明
pick 0d4a808 添加pull的说明

# Rebase 326fc9f..0d4a808 onto d286baa
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

将第一行的 pick 改成 edit，然后保存并退出。可以修改一下内容文件，比如 sample.txt 之类的。

```bash
$ git add sample.txt
$ git commit --amend
$ git rebase --continue
```

这时，有可能其他提交会发生冲突, 请修改冲突部分后再执行 `add` 和 `rebase --continue`。这时不需要提交。如果在中途要停止 `rebase` 操作，请在 `rebase` 指定 `--abort` 选项执行，这样就可以抹去并停止在 `rebase` 的操作。

实际上，在 rebase 之前的提交会以 ORIG_HEAD 之名存留。如果 rebase 之后无法复原到原先的状态，可以用 `git reset --hard ORIG_HEAD` 复原到 rebase 之前的状态。

### 延伸阅读:

[git commit 合并](http://www.douban.com/note/318248317/)
