---
layout: post
title:  "Vim Plugin"
date:   2014-12-16 10:00:00
categories: tool
tags: editor
author: "Victor"
---

## 基础插件

### NERDTree

定位到 NERDTree 之后按 ```?``` 来查看帮助信息。

* t 在新tab页面文件
* o 标准在当前窗口打开当前文件或文件夹
* s 新建水平窗口打开当前文件
* i 新建垂直窗口打开当前文件
* p 回到上级目录
* m 显示操作菜单

### tcomment

多行注释

* gc 选中多行之后，注释或取消注释
* gc{motion} motion 可以是一组移动动作，例如 5j
* gcc 当前行的，注释或取消注释


### Tagbar

显示方法列表

* <CR> 回车在编辑窗口打开当前方法
* p 跳转到该方法，但焦点扔在 TagBar

### ctrlp.vim

* ctrl + p 打开查找文件的功能
* ctrl + d 切换文件名查找模式 - 推荐
* ctrl + r 切换正则查找模式
* ctrl + j | k 在搜索结果中上下切换
* ctrl + t | v | x 分别在新的 tab 页面或者分割窗口中打开选中的文件
* 按回车直接在当前窗口中打开选中文件
* 更多可以参考 http://kien.github.io/ctrlp.vim/

### YouCompleteMe

安装起来还是比较麻烦。

先编辑 ```~/.vimrc```

```
syntax on
set clipboard=unnamed
set number
set tabstop=4 shiftwidth=4 expandtab

set nocompatible
filetype off

set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

Bundle 'gmarik/vundle'
Bundle 'Valloric/YouCompleteMe'

filetype plugin indent on
```

在命令行下运行

```
git clone https://github.com/Valloric/YouCompleteMe.git ~/.vim/bundle/YouCompleteMe
cd ~/.vim/bundle/YouCompleteMe
git submodule update --init --recursive
./install.sh --clang-completer
```

最后运行下面的命令

```
vim +BundleInstall +qall
```

最后我把这个插件删除了，因为它只能在 macvim 下使用。

### neocomplete neosnippet neosnippet-snippets

这三个插件是跟自动补全相关的

安装方法是在 .vimrc 下面添加如下配置并

```
Bundle 'Shougo/neocomplete'
Bundle 'Shougo/neosnippet'
Bundle 'Shougo/neosnippet-snippets'
```

* 用 ctrl + n | p 来选择
* 用 tab 键来确认所选
