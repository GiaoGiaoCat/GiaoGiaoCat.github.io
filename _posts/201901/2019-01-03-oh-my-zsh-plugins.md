---
layout: post
title:  "使用 Antigen 管理 Oh-My-Zsh Plugins"
date:   2019-01-03 12:00:00

categories: tool
tags: terminal
author: "Victor"
---

## 简介

Oh-My-Zsh 可以很方便的让 ZSH 的新手能有一套还不错的起步，而且提供了丰富插件和配色方案。但是它的插件管理一直是个老大难问题。

我们可以使用 Antigen 和 Antibody 来管理 ZSH 的插件。他们的灵感来自 Vim 的插件管理方式。

## 开始使用 Antigen

### 插件安装

1. `brew install autojump`
2. `brew install thefuck`
3. 安装 colorls，直接参考 [带图标的 ls 命令](/tool/color-ls/)
4. `brew install antigen`

### 配置

```bash
# The plugin manager for zsh.
source /usr/local/share/antigen/antigen.zsh

# Load the oh-my-zsh's library.
antigen use oh-my-zsh

# Load the theme
antigen theme https://github.com/denysdovhan/spaceship-zsh-theme spaceship

# Bundles from the default repo (robbyrussell's oh-my-zsh).
antigen bundle autojump
antigen bundle brew
antigen bundle brew-cask
antigen bundle bundler
antigen bundle common-aliases
antigen bundle colored-man
antigen bundle git
antigen bundle gitfast
antigen bundle git-extras
antigen bundle git-flow
antigen bundle rails
antigen bundle rake
antigen bundle ruby
# antigen bundle osx
antigen bundle safe-paste
antigen bundle vi-mode

antigen bundle zsh-users/zsh-autosuggestions

# Syntax highlighting bundle.
antigen bundle zsh-users/zsh-syntax-highlighting
antigen bundle zsh-users/zsh-history-substring-search ./zsh-history-substring-search.zsh

# Tell Antigen that you're done.
antigen apply

# Magnificent app which corrects your previous console command.
eval $(thefuck --alias)

# https://github.com/athityakumar/colorls
source $(dirname $(gem which colorls))/tab_complete.sh
```

## 相关阅读

* [SNAPS](https://snapcraft.io/)
* [Antibody](http://getantibody.github.io/)
* [Antigen](https://github.com/zsh-users/antigen/)
* [Managing Oh My Zsh Using Antigen](https://amitd.co/blog/managing-oh-my-zsh-using-antigen)
* [Terminals Are Sexy](https://terminalsare.sexy/)
* [awesome-zsh-plugins](https://project-awesome.org/unixorn/awesome-zsh-plugins)
* [awesome-shell](https://github.com/alebcay/awesome-shell)
* [awesome-macos-command-line](https://github.com/herrbischoff/awesome-macos-command-line)
