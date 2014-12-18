---
layout: post
title:  "git 用法合集"
date:   2014-08-29 11:58:00
categories: tool
tags: git
author: "Victor"
---

### 善用 .gitignore 忽略某些文件

```bash
# 以'#' 开始的行，被视为注释.
# 忽略掉所有文件名是 foo.txt 的文件.
foo.txt
# 忽略所有生成的 html 文件,
*.html
# foo.html是手工维护的，所以例外.
!foo.html
#  忽略所有.o 和 .a文件.
*.[oa]
```

### 设置全局代理

```bash
git config --global http.proxy 'socks5://127.0.0.1:7070'
```

也可手动修改 ~/.gitconfig 文件中的配置

```bash
[http]
proxy = socks5://127.0.0.1:12441
```

### clone 项目的时候只拉取最后一次的提交结果

--depth=1 是指只拉取最后一次的提交结果。

```
git clone git@github.com:laravel/laravel.git --depth=1
```

### 如何取消全部的本地修改

如果你没有提交可以直接用 ```git checkout .``` 如果提交了需要先用 **reset** 命令回退再执行 ```git checkout . ```


### 暂存当前修改已切换到其它分支

这个命令不会保存当前分支中Untracked的文件，所以记得在切到其他分支的时候，谨慎使用git clean

```bash
# 临时保存当前分支的修改
git stash
# 更复杂点
git stash save [--keep-index] [<message>]
# 列出所有的stash
git stash list
# 恢复
git stash apply
# or
git stash pop
```

###git stash recover 恢复

最简单的用法就是

```bash
git stash
do some work
git stash pop

git stash list #将当前的Git栈信息打印出来
git stash clear #将栈清空
git stash apply stash@{1}’ #就可以将你指定版本号为stash@{1}的工作取出
```

git stash 默认不会把untracked files加进去，如果希望把不在版本库的文件也加进去，如何做？

```bash
git stash -u
```

git pop 后refs会被dropped掉，我们会看见如下的信息：

```bash
Dropped refs/stash@{0} (f798acc46e0838e5c826d177124ab95a73ac92ca)
```

如何恢复？很简单，如果我们能得到那串SHA1的值，直接

```bash
git stash apply f798acc46e0838e5c826d177124ab95a73ac92ca
```

如果我们找不到SHA1的值了，可以在git的–lost-found中找到：

```bash
git fsck --no-reflog | awk '/dangling commit/ {print $3}'
```

拿到SHA1的值只要再apply就可以了

### 参考

* [git 小技巧](https://github.com/xiaobeicn/programming-skills-summary/tree/master/git)
