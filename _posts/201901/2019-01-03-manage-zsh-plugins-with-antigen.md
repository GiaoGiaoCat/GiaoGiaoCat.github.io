---
layout: post
title:  "使用 Antigen 管理 Zsh Plugins"
date:   2019-01-03 12:00:00

categories: tool
tags: terminal
author: "Victor"
---

## 简介

Oh-My-Zsh 可以很方便的让 ZSH 的新手能有一套还不错的起步，而且提供了丰富的插件和配色方案。但是它的插件管理一直是个老大难问题。

关于 Oh-My-Zsh 的更多吐槽可以直接看这篇文章，[Oh-my-zsh is the Disease and Antigen is the Vaccine](https://joshldavis.com/2014/07/26/oh-my-zsh-is-a-disease-antigen-is-the-vaccine/)。

## 开始使用 Antigen

Antigen 的灵感来自 Vim 的包管理器 Vundle。你需要做的就是在 `.zshrc` 中列出插件，之后就会自动安装它们。和其它的包管理软件一样， Antigen 提供命令让我们更新或删除插件。

### 安装

先装好各种依赖。

1. `brew install autojump thefuck bash-completion git-extras`
2. 安装 colorls，直接参考 [带图标的 ls 命令](/tool/color-ls/)
3. `brew install antigen`

### 配置

```bash
# The plugin manager for zsh.
source /usr/local/share/antigen/antigen.zsh

# Load the oh-my-zsh's library.
antigen use oh-my-zsh

# Load the theme
antigen theme https://github.com/denysdovhan/spaceship-zsh-theme spaceship
SPACESHIP_PROMPT_FIRST_PREFIX_SHOW=true
SPACESHIP_TIME_SHOW=true
SPACESHIP_TIME_PREFIX="["
SPACESHIP_TIME_SUFFIX="] "
SPACESHIP_TIME_FORMAT="%D{百分号Y-%m-%d} %*"
SPACESHIP_DIR_TRUNC_REPO=false
SPACESHIP_DIR_PREFIX="["
SPACESHIP_DIR_SUFFIX="] "

# Bundles from the default repo (robbyrussell's oh-my-zsh).
antigen bundle autojump
antigen bundle brew
antigen bundle brew-cask
antigen bundle bundler
antigen bundle common-aliases
antigen bundle colored-man
antigen bundle extract
antigen bundle gitfast
antigen bundle git-extras
antigen bundle rails
antigen bundle rake
antigen bundle ruby
antigen bundle safe-paste
antigen bundle sublime
antigen bundle thefuck
antigen bundle vi-mode

# For SSH, starting ssh-agent is annoying
antigen bundle ssh-agent

# Plugins not part of Oh-My-Zsh can be installed using githubusername/repo
antigen bundle zsh-users/zsh-autosuggestions
# antigen bundle zsh-users/zsh-history-substring-search ./zsh-history-substring-search.zsh
antigen bundle zsh-users/zsh-syntax-highlighting

# Tell Antigen that you're done.
antigen apply

# User configuration
export PATH="/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:$PATH:$HOME"
MYSQL="/usr/local/mysql/bin"
export PATH="$PATH:$MYSQL"
export DYLD_LIBRARY_PATH="/usr/local/mysql/lib:$DYLD_LIBRARY_PATH"
export EDITOR='subl'

# rbenv
eval "$(rbenv init -)"

# https://github.com/athityakumar/colorls
source $(dirname $(gem which colorls))/tab_complete.sh

# http://stackoverflow.com/questions/4975973/gem-update-unable-to-convert-xe7-to-utf-8-in-conversion-from-ascii-8bit-to-u
export LC_CTYPE=en_US.UTF-8

# Load custom aliases and functions
source $HOME/.oh-my-zsh/custom/victor.zsh
```

### 其它命令

```
antigen help
antigen update # 更新插件
antigen revert # 回退到更新插件之前
antigen list [--simple|--short|--long] # 列出所有安装的插件
antigen cleanup # 清理掉所有当前未使用的插件
antigen purge example/bundle [--force] # 从文件系统上删除插件
antigen reset # 清除生成的缓存，这个命令经常与 antigen init 配合使用
antigen use # 加载 zsh 框架，比如 oh-my-zsh 和 prezto
antigen theme # 切换提示符的主题
antigen apply # 所有之前所做的更改
```

### 介绍一些 zsh 插件和命令

* zsh-autosuggestions 根据历史记录即时提示，brew 不用安装配方就能用
* zsh-syntax-highlighting 给命令添加颜色，brew 不用安装配方就能用
* autojump 是一款 [FS Jumping](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins-Overview#fs-jumping) 工具，无脑使用 `j` 跳跃进入各种目录
* extract 解压插件，直接使用 `x xxx.rar` 就行，不用记命令
* `d` 按回车，看到最近的历史记录，再输入数字，可以执行那条命令
* `zsh_stats` 使用频率前 20 的命令是什么
* `take` 看看 `which take` 就知道它有什么用了
* `clipcopy` 和 `clippaste` 剪贴板和命令行的交互
* `...`, `....`
* [官方 wiki](https://github.com/robbyrussell/oh-my-zsh/wiki)

## 相关阅读

* [SNAPS](https://snapcraft.io/)
* [Antibody](http://getantibody.github.io/)
* [Antigen](https://github.com/zsh-users/antigen/)
* [spaceship-prompt](https://github.com/denysdovhan/spaceship-prompt)
* [Managing Oh My Zsh Using Antigen](https://amitd.co/blog/managing-oh-my-zsh-using-antigen)
* [Terminals Are Sexy](https://terminalsare.sexy/)
* [awesome-zsh-plugins](https://project-awesome.org/unixorn/awesome-zsh-plugins)
* [awesome-shell](https://github.com/alebcay/awesome-shell)
* [awesome-macos-command-line](https://github.com/herrbischoff/awesome-macos-command-line)
