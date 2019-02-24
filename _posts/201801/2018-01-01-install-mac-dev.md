---
layout: post
title:  "同步开发环境到一台全新的电脑"
date:   2018-01-01 1:30:00
categories: tool
tags: mac
author: "Victor"
---

用了将近 10 年的 Mac 终于在 2017 年底的倒数第2天买了第一台 MacbookPro。怎么样一边玩游戏一边把电脑开发环境配置好呢？

开机之后先登录 MAS 更新系统补丁，安装Xcode 或 `xcode-select --install`，搞定 ClashX。

## Preferences

* `Keyboard > Text >` Disable "Correct spelling automatically".
* `Security and Privacy > Firewall >` On
* `Security and Privacy > General >` App Store and identified developers
* `File Sharing >` Off
* `Trackpad >` Tap to click
* `Keyboard > Key Repeat >` Fast (all the way to the right)
* `Keyboard > Delay Until Repeat >` Short (all the way to the right)
* `Keyboard > Shortcuts > Input Sources` Select the previous input source `CMD + 空格`，其它禁用
* `Keyboard > Shortcuts > Spotlight` Show Spotlight Search `ALT + CMD + 空格`，其它禁用
* `Keyboard > Shortcuts` 选择下一个输入法使用 `CMD + 空格`，其它禁用
* `Dock >` Automatically hide and show the Dock

## DropBox

先配好全局跳墙，然后去 DropBox 官网下载客户端并登录同步自己的文件夹。这里有 Mackup 的全部配置。

## Homebrew & Zsh Plugins

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew tap homebrew/bundle
brew tap caskroom/cask
```

如果开局已经登录 MAS 下面这步就可以忽略了。

```bash
brew install mas
mas signin email@email.com
```

执行 `brew bundle exec -- bundle install` 搞定全部软件。

[这里](https://gist.github.com/wjp2013/66dbd649203e822eb6da110300fead47) 是我的 Brewfile。

按照[这篇文章](/tool/manage-zsh-plugins-with-antigen/)把 Zsh Plugins 配置好，这一步会同时把字体配置好。

## Mackup

把软件配置都同步过来。

```bash
mackup restore
```

## SpaceVim

```bash
curl -sLf https://spacevim.org/install.sh | bash
```

## Dock && Menu

按照原来的电脑把 Dock 上的图标和 Menu 上的图标都排列好，正版软件都激活。

## iTerm

* Set hotkey to open and close the terminal to `command + option + i`
* Go to `profiles -> Default -> Terminal` Check `silence bell` to disable the terminal session from making any sound
* Go to `General -> Preferences` Check `Load preferences from a custom folder or URL:` 选 `~/Dropbox/Apps/iterm2`
* 修改 Profile 之后可以手动点 `General -> Preferences -> Save settings to Folder`
* Download Mediallion theme

```bash
# Specify the preferences directory
defaults write com.googlecode.iterm2.plist PrefsCustomFolder -string "~/Dropbox/Apps/iTerm2"
# Tell iTerm2 to use the custom preferences in the directory
defaults write com.googlecode.iterm2.plist LoadPrefsFromCustomFolder -bool true
```

## rbenv

确保安装了 `brew install readline`

```
RUBY_CONFIGURE_OPTS=--with-readline-dir="$(brew --prefix readline)" rbenv install 2.5.0
```

## Other

1. VSCode `Sync:download settings` 同步已经安装的插件和快捷键等配置
2. Alfred `defaults write com.runningwithcrayons.Alfred-Preferences-3 dropbox.allowappsfolder -bool TRUE` 然后修改 General 中的 sync 文件夹到 `~/Dropbox/Apps/Alfred`
3. AirDrop 把 `~/.ssh` 中的 `id_rsa` 和 `id_rsa.pub` 拿过来，所有电脑用同一套 ssh key
4. 网页登录各种账户 chrome，github, things, spotify 什么的
5. 如果有些词打不出来，可以更新一下词库 https://github.com/rime-aca/dictionaries
6. `defaults write com.sublimetext.3 ApplePressAndHoldEnabled -bool false` holding down a key won't repeat it
7. `sudo spctl --master-disable` 启用安装任何来源的应用

## 其它软件的安装

* Kaleidoscope
* mubu

## 相关链接

* https://github.com/taniarascia/mac
* https://github.com/sb2nov/mac-setup
* https://github.com/nicolashery/mac-dev-setup
* https://mallinson.ca/osx-web-development/
* https://www.taniarascia.com/setting-up-a-brand-new-mac-for-development/
* https://github.com/Homebrew/homebrew-bundle
* https://robots.thoughtbot.com/brewfile-a-gemfile-but-for-homebrew
* https://github.com/sharat/vscode-brewfile
* https://www.sublimetext.com/docs/3/vintage.html
