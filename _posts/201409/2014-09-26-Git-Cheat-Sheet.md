---
layout: post
title:  "Git Cheat Sheet Chinese"
date:   2014-09-26 10:00:00
categories: tool
tags: git
author: "Victor"
---

## 基础概念

* `pull` = `fetch + merge`
* `HEAD` 指向的是现在使用中的分支的最后一次更新。通常默认指向 master 分支的最后一次更新。通过移动 HEAD，就可以变更使用的分支。
* `~` 指定HEAD之前的提交记录，`^` 指定使用哪个为根节点。
* `fast-forward` 合并 bugfix 分支到 master 分支时，master 状态没有被改过，通过把 master 分支的位置移动到 bugfix 的最新分支上，Git 就会合并。这样的合并被称为 fast-forward（快进）合并。


![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2014-09-26-Git-Cheat-Sheet/capture_stepup1_3_2.png)

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2014-09-26-Git-Cheat-Sheet/capture_stepup1_4_1.png)


## 创建

```bash
# 复制一个已创建的仓库:
$ git clone ssh://user@domain.com/repo.git

# 创建一个新的本地仓库:
$ git init
```

## 本地修改

```bash
# 显示工作路径下已修改的文件：
$ git status

# 显示与上次提交版本文件的不同：
$ git diff

# 把当前所有修改添加到下次提交中：
$ git add .

# 把对某个文件的修改添加到下次提交中：
$ git add -p <file>

# 提交本地的所有修改：
$ git commit -a

# 提交之前已标记的变化：
$ git commit

# 附加消息提交：
$ git commit -m 'message here'

# 修改上次提交
# Don't amend published commits!
$ git commit --amend

# 移除文件并连带从工作目录中删除文件，以后不会出现在未跟踪文件清单中
$ git rm <resolved-file>
# 从仓库中删除，但仍然保留在当前工作目录中。
$ git rm --cached <resolved-file>
```

## 提交历史

```bash
# 从最新提交开始，显示所有的提交记录（显示hash， 作者信息，提交的标题和时间）：
$ git log

# 查看最后一次提交及其改动：
$ git log 1 -p

# 一行显示一个提交：
$ git log --pretty=oneline

# 以树状图显示提交历史：
# --graph 能以文本形式显示更新记录的流程图, --oneline 能在一行中显示提交的信息，--decorate 显示包含标签。
$ git log --graph --oneline --decorate --all

# 显示所有提交（仅显示提交的hash和message）：
$ git log --oneline

# 显示某个用户的所有提交：
$ git log --author="username"

# 显示某个文件的所有修改：
$ git log -p <file>

# 谁，在什么时间，修改了文件的什么内容：
$ git blame <file>
```

## 分支与标签

```bash
# 列出所有的分支：
$ git branch

# 切换分支：
$ git checkout <branch>

# 基于当前分支创建新分支：
$ git branch <new-branch>

# 基于远程分支创建新的可追溯的分支：
$ git branch --track <new-branch> <remote-branch>

# 删除本地分支:
$ git branch -d <branch>

# 给当前版本打轻标签：
$ git tag <tag-name>

# 给当前版本打注解标签：
$ git tag -a <tagname>
$ git tag -am "some text" <tagname> # -m 选项添加注解
$ git tag -n # 显示标签的列表和注解
$ git tag -d <tagname> # 删除标签
```

## 更新与发布

```bash
# 列出对当前远程端的操作：
$ git remote -v

# 显示远程端的信息：
$ git remote show <remote-name>

# 添加新的远程端：
# 可以给远程数据库取一个别名 <remote-name>，下次推送的时候直接使用别名就好。
# 推送或者拉取的时候，如果省略了远程数据库的名称，则默认使用名为 origin 的远程数据库。因此一般都会把远程数据库命名为 origin。
$ git remote add <remote-name> <url>
$ git remote add origin git@github.com:[your_space_id]/[your_project_key].git

# 下载远程端版本，但不合并到HEAD中：
$ git fetch <remote-name>

# 下载远程端版本，并自动与HEAD版本合并：
$ git remote pull <remote-name> <url>

# 将远程端版本合并到本地版本中：
$ git pull origin master

# 将本地版本发布到远程端：
# 指定了`-u` 选项，那么下一次推送时就可以省略分支名称了。
# 首次运行指令向空的远程数据库推送时，必须指定远程数据库名称和分支名称。
$ git push origin <remote-name> <branch>

# 删除远程端分支：
git push <remote-name> --delete <branch> (since Git v1.7.0)

# 发布标签:
$ git push --tags
```

## 合并与重置

```bash
# 将分支合并到当前HEAD中：
$ git merge <branch>
$ git merge --squash <branch> # 汇合主题分支的提交，然后合并提交到目标分支

# 将当前HEAD版本重置到分支中:
# Don't rebase published commit!
$ git rebase <branch>

# 退出重置:
$ git rebase --abort

# 解决冲突后继续重置：
$ git rebase --continue

# 使用配置好的merge tool 解决冲突：
$ git mergetool

# 在编辑器中手动解决冲突后，标记文件为`已解决冲突`
$ git add <resolved-file>
```

## 撤销

```bash
# 放弃工作目录下的所有修改：
$ git reset --hard HEAD

# 移除缓存区的所有文件（i.e. 撤销上次`git add`）:
$ git reset HEAD

# 放弃某个文件的所有本地修改：
$ git checkout HEAD <file>

# 重置一个提交（通过创建一个截然不同的新提交）
$ git revert <commit>

# 将HEAD重置到上一次提交的版本，并放弃之后的所有修改：
$ git reset --hard <commit>

# 放弃工作目录下最后两次的所有修改
$ git reset --hard HEAD~~

# 将HEAD重置到上一次提交的版本，并将之后的修改标记为未添加到缓存区的修改：
$ git reset <commit>

# 将HEAD重置到上一次提交的版本，并保留未提交的本地修改：
$ git reset --keep <commit>

# 在 reset 之前的提交可以参照 ORIG_HEAD，Reset 错误的时候，在 ORIG_HEAD 上 reset 就可以还原到 reset 前的状态
$ git reset --hard ORIG_HEAD

# 将本地仓库重置成与远端一样：
$ git fetch origin git reset --hard origin/master
```

## 暂存

```bash
$ git stash
$ git stash list
$ git stash pop
```

## 原文链接

* [Git Cheat Sheet](https://github.com/flyhigher139/Git-Cheat-Sheet)
* [git - 简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)
* [猴子都能懂的GIT入门](https://backlog.com/git-tutorial/cn/intro/intro1_1.html)
