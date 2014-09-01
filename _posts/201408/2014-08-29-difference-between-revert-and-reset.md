---
layout: post
title:  "git revert 和 git reset 的区别"
date:   2014-08-29 11:55:00
categories: tool
tags: git
author: "Victor"
---

## revert
`git revert` 撤销某次操作，此次操作之前和之后的 commit 和 history 都会保留，并且把这次撤销作为一次最新的提交。git revert是提交一个新的版本，将需要revert的版本的内容再反向修改回去，版本会递增，不影响之前提交的内容。

### 没有push
当代码已经 commit 但是还没有 push 的时候。可以使用下面的命令，

```ruby
git revert HEAD #撤销倒数第一次提交
git revert HEAD^ #撤销倒数第二次提交
git-revert HEAD~2 #撤销倒数第三次提交
git revert commit  #（比如：fa042ce57ebbe5bb9c8db709f719cec2c58ee7ff）撤销指定的版本，撤销也会作为一次提交进行保存。
```

撤销之后，会造成代码冲突，需要手动解决冲突然后提交代码。
假如本地有2条 commits没有 push。执行`git revert HEAD~1`会把Working Copy回退到服务器上的前一个版本。

### 已经push
当代码已经 commit 并且 push。使用下面的命令，

```ruby
git revert HEAD~1 #代码回退到前一个版本
```

然后需要手动合并冲突并进行修改，再 commit 和 push。这相当于增加了一次新的提交并且版本库中有记录。


## reset
`git reset` 是撤销某次提交，但是此次之后的修改都会被退回到暂存区。

如果我们的有两次 commit 但是没有 push 代码。

```ruby
git reset HEAD~1 #撤销前一次 commit，所有代码回到 Working Copy
```

假如我们有几次代码修改，并且都已经 push 到了版本库中。

```ruby
git reset --hard HEAD~2 #本地的Wroking Copy回退到2个版本之前。
```

现在操作你需要修改的文件，然后重新 commmit。在 push 之前我们需要先 pull 代码下来。这会造成两个版本的冲突。代码会类似

```
Yeah
Ok Ok
<<<<<<< HEAD

I am superman.
Really?
=======
Let's go
>>>>>>> 04831502235d60482f31652d3529bb7042807e21
```

当然可以手动解决冲突，或者直接选择使用本地的文件或者服务端的文件。然后 push 代码。如果代码 push 失败，需要在服务器的 master 分支下进行代码合并（Merge）。

### 只回退某个指定文件到指定版本

```bash
git reset a4e215234aa4927c85693dca7b68e9976948a35e MainActivity.java
```

### 延伸阅读

* [Git常用命令备忘](http://robbinfan.com/blog/34/git-common-command)
* [Rebase用法](http://gitbook.liuhui998.com/4_2.html)
