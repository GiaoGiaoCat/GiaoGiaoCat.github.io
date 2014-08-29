---
layout: post
title:  "提高 Shell 和 Vim 的工作效率"
date:   2014-08-29 14:47:00
categories: tool
tags: terminal editor
author: "Victor"
---

本文假设您已经有一定操作 shell 的经历，并且会简单的 vim 操作。

### Remap Your CAPSLOCK Key

把 CAPSLOCK 键映射成 Ctrl 键

### Use ctrl-r For History Autocomplete

在 Terminal 中学会使用 ```Ctrl + r``` 来自动补全历史记录中的命令

### Switch to iTerm on OS X

iTerm 是一个为 Mac OS X 编写的，功能齐全的终端仿真程序。建议用 **iTerm2** 替代 **Terminal** 。

它的几个常用快捷键：

* Cmd + f 查找。支持正则。其中查找的内容会被自动复制
* Cmd + shift + d 水平分屏
* 使用 Cmd + ] 和 Cmd + [ 在最近使用的分屏直接切换
* Cmd + ； 自动补全历史命令

几篇文章：

* [iTerm2新手应知特色功能](http://www.yangzhiping.com/tech/iterm2.html)
* [fooCoder的iTerm2使用之道](http://www.macx.cn/thread-2098800-1-1.html)
* [我的iTerm2+tmux](http://www.7lemon.net/Tools/2012/09/16/iterm2tmux/)

### Switch to Zsh

好处如下：

* 支持自定义路径缩写
* 拼写单词检查
* 共享命令历史记录
* 一些语法高亮支持
* 支持 vim 模式
* 支持 OhMyZsh


### Remap your Vim Escape Key

修改 **.vimrc** 文件 添加 ```inoremap jj <ESC>```

### Remap your Vim Leader Key

```
nnoremap j VipJ
let mapleader = “,”
```
### Use vi-mode in Your Shell

修改 **.zshrc**

```
bindkey -v
bindkey -M viins 'jj' vi-cmd-mode
bindkey '^R' history-incremental-search-backward
```

### Add tmux to Your Workflow

tmux是一个优秀的终端复用软件。使用它最直观的好处就是，通过一个终端登录远程主机并运行tmux后，在其中可以开启多个控制台而无需再“浪费”多余的终端来连接这台远程主机。

* [tmux 百度百科](http://baike.baidu.com/view/9065064.htm)
* [A tmux Tutorial and Primer](http://www.danielmiessler.com/study/tmux/)
* [ITerm2 TMux and Vim Setup](http://candland.net/2011/11/08/iterm2-tmux-and-vim-setup/)
* [Swap iTerm2 cursors in vim insert mode when using tmux](https://gist.github.com/andyfowler/1195581)

### Synchronize Your Environments

创建一个 git 版本库，把 bash, .vimrc, .zshrc 等配置文件都纳入版本控制。或者用 dropbox 来备份。


### 相关文章

* [9 Enhancements to Shell and Vim Productivity](http://www.danielmiessler.com/blog/enhancements-to-shell-and-vim-productivity)
