---
layout: post
title:  "git 删除错误提交的 commit"
date:   2014-08-29 11:41:00
categories: tool
tags: git
author: "Victor"
---

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



又一次: 把 master 的提交路线搞成了狗屎

![](https://raw.githubusercontent.com/wjp2013/the_room_of_exercises/master/assets/wiki/001.jpg)

方法:

1. 到 master 分支利用命令 ```git reset <commit_id>```
2. git push origin HEAD --force

这里的 commit_id 要用分叉之前的，这样所有的代码不会丢，develop 分支上的提交信息也不会丢。那些奇怪的 merge 信息就没了。


### 延伸阅读:

[git commit 合并](http://www.douban.com/note/318248317/)
