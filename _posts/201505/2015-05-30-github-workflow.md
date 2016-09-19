---
layout: post
title:  "Understanding the GitHub Flow"
date:   2015-05-30 14:00:00
categories: tool
tags: git
author: "Victor"
---

GitHub Flow 是一个轻量级，基于分支的工作流程。它让团队和项目的开发变得更有规律。本指南解释了 GitHub Flow 是如何工作的。

## Create a branch

在项目的进程中，我们随时都会有一堆不同的功能和想法，其中的一些正准备发布，而另外一些还在开发中。分支的存在可以帮你管理这个工作流程。

创建一个分支意味着我们创建了一个新的环境用来试验一些新想法。在新分支上工作不会影响 master 分支，你可以放心的进行试验并提交修改。通常在你的同事们帮你审查代码前，分支是不会被合并的。

### ProTip

分支是 Git 的核心概念，GitHub Flow 也是基于它。唯一的原则是：master 分支任何时候都是可部署的。

当开发一个新功能或者修复一个 bug 的时候，我们的新分支是基于 master 创建的。分支名称应该是描述性质的，例如：`refactor-authentication, user-content-cache-key, make-retina-avatars`，这样其他同事就知道你目前正在做什么。

```bash
git checkout master
git pull origin master
git checkout -b <my branch name>
```

## Add commits

分支创建后，我们开始在其上进行开发。每当你添加，修改或删除文件，都需要进行 `commit` ，并推送到分支。这些 `commits` 可以让我们追踪该分支上的工作过程。

`Commits` 同样是一个透明的历史记录，方便同事们理解你是如何完成这个任务的。每个 `commit` 要有一个 `Commit messages`，用来解释本次提交的细节。此外，每次提交应该是一个独立的单元。这样当我们发现 bug 的时候，可以方便的回滚代码，并且选择另外的解决方案。

### ProTip

`Commit messages` 是非常重要的，当你把 `commit` 推送到服务器上后，Git 会跟踪你的修改并把这些信息显示出来。清晰的 `Commit messages`，可以让你和同事更容易的理解开发进度并进行反馈。

经常在工作分支上执行 `git rebase -i master`，将 master 上的最新的代码合并到当前分支上，这里的 `-i` 的作用是将我们当前分支之前的 commit 压缩成为一个。这样做的好处在于当我们之后创建 PR 并进行相应的 Code Review 的时候，代码的改动会集中在一个 commit，使得 Code Review 更直观方便

## Open a Pull Request

`Pull Requests` 会创建关于 `commits` 的讨论。因为它们和 Git 仓库结合在一起，所以任何人都能清晰的看到什么样的变化将会被合并，以决定是否接受你的请求。

你可以在开发的过程中的任何时候创建一个 `Pull Request`：不论你是否改动代码，如果想分享一些截图或者一些想法；当你被卡主需要帮助和建议；又或者当你已经完成功能需要大家对你的工作做一些审核。利用 Github 的 `@` 功能，在你的 `Pull Request` 消息中，你可以请求特定的人和团队给你反馈，不管他们在不在线，在哪个时区。

### ProTip

`Pull Requests` 尤其适用于开源项目的贡献和共享资源库的管理。如果你使用 `Fork & Pull` 模型，`Pull Requests` 提供了一种方法来通知项目的维护者，你有一些希望他们注意到的修改。如果你使用的是 `Shared Repository` 模型，`Pull Requests` 帮助你在合并分支进入 master 之前进行代码审查和基于这些代码的建议、讨论。

## Discuss and review your code

打开一个 `Pull Request` 后，同事和团队在审查你的修改时会有一些问题和讨论。或许是因为代码风格不符合团队规范，或许是缺少单元测试，也许一切看起来不错只是顺序有点问题。`Pull Requests` 设计的目的就是为了鼓励这样的讨论。

你可以持续推送分支，以获得关于这些提交的讨论和反馈。如果有人评论说你忘记做了某个任务，或者在代码中出现错误，那么可以在这个分支中解决并推送修改。Github 会在 `Pull Request` 视图中显示新的提交和追加的评论。

### ProTip

`Pull Request` 评论支持 Markdown 格式，所以你可以嵌入图片和 emoji，格式化的代码块和其它轻量级的格式。

## Merge and deploy

一旦 `Pull Request` 通过审查和测试，就可以把它合并进入 master 分支进行部署。如果你想要在 GitHub 版本库合并前进行一些测试，你可以可以先在本地合并。

一旦合并，`Pull Requests` 会把你的代码修改保存一条历时记录。因为它是可搜索的，所以任何人都能通过回溯来了解到你是如何完成该功能的。

### ProTip

通过 `Pull Requst` 的某些关键词，你可以把代码和 issue 关联起来。当 `Pull Requst` 合并后，相关的 issue 也会被关闭。例如：输入 `Closes #32` 会关闭该 issue。更多的功能请查看[帮助文档](https://help.github.com/articles/closing-issues-via-commit-messages)。

## 相关阅读

* [Git 风格指南](https://github.com/aseaday/git-style-guide)
* [Understanding the GitHub Flow](https://guides.github.com/introduction/flow/)
* [GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html)
* [GitHub Cheat Sheet](https://github.com/tiimgreen/github-cheat-sheet)
* [Git Flow vs Github Flow](http://lucamezzalira.com/2014/03/10/git-flow-vs-github-flow/)
