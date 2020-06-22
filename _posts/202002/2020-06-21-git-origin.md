---
layout: post
title:  "为 git 同时添加两个远程仓库"
date:   2020-06-18 17:10:00

categories: tool
tags: git
author: "Victor"
---

虽然主要代码都托管在 Github 中，但是因为阿里云的云效访问 Github 太慢，所以打算把仓库同步到 https://codeup.aliyun.com/ 上。

## 多地址的 remote repo

一般来说最佳方法是给 origin 设两个地址：

```bash
git remote set-url origin --add https://github.com/USERNAME/REPO1.git
git remote set-url origin --add https://github.com/USERNAME/REPO2.git
```

在 .git/config 里得到

```yaml
...
[remote "origin"]
    url = https://USERNAME@github.com/USERNAME/REPO1.git
    url = https://USERNAME@github.com/USERNAME/REPO2.git
...
[branch "master"]
    remote = origin
...
```

然后 `git push origin master` 就会同时提交到两个 repo，而 `git pull origin master` 会从两个 repo 里取得更新。

当然 URL 和 repo 不一定非要是 GitHub 上的，具有两个 url 的 remote 也不一定要是 origin，比如可以设置成 all。

## 只用于 push 的备份 repo

你想从 repo1 pull，但是 push 的时候要推送到 repo1 和另一个 repo2。

```bash
git remote set-url origin --add https://github.com/USERNAME/REPO1.git
git remote set-url origin --push --add https://example.com/USERNAME/REPO2.git
```

在 .git/config 里得到

```yaml
...
[remote "origin"]
    url = https://USERNAME@github.com/USERNAME/REPO1.git
    pushurl = https://USERNAME@example.com/USERNAME/REPO2.git
...
[branch "master"]
    remote = origin
...
```

然后 `git push origin master` 就会同时提交到两个 repo，而 `git pull origin master` 会从两个 GitHub repo1 里取得更新。

## 原文

* [为git同时添加两个远程仓库](https://www.cnblogs.com/huipengly/articles/8306855.html)
