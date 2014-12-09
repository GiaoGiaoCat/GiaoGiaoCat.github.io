---
layout: post
title:  "Venture Into Vim"
date:   2014-11-24 10:10:00
categories: tool
tags: editor
author: "Victor"
---

## Introduction

### Basic Movement

```
h, j, k, l
:w welcome.txt
:q
```

### Faster Movement

```
w, b, W, B, $, ^, 0, gg, G, {, }, f(*), F(*), t(*), T(*), (*)gg, (*)G, :(*)
3w, 2fb, tf, 15G
```

### Basic Editing

```
x, X, u, d(*), c(*), (*)dd, (*)cc, (*)i(*), (*)a(*),
ct", ci", ca'
```

### Cut, Copy and Paste

```
p, P, y(*), yy
10p
```

## Digging Deeper

### Search

```
/, ?, n, /., :noh
/.[aeiou], ?\n, d/(, c?d, y/)
```

### Replace

* 在 visual mode 下，s 仅替换选中区域
* gv 选中上一个 visual mode 下的区块
* gci 的意思是 global, confirm, inside, 也就是可以替换单词中的匹配字符
* \zs 指明匹配由此开始 :help /\zs, 同样也有 \ze
* 替换的流程是，先 /匹配，然后 visual mode 选取区块，然后 :s//replacement 替换

```
:s, (*)g, shift + v, %, :help (*), gv
:%s/pattern/replacement/gci, :s/pattern/replacement/, v%, :help :s, /[^ ](
```

想要验证自己是否掌握替换的用法，可以尝试能把下面的代码块

```
funciont()
// do something
function() {
  alert("Hello")
}
``

替换成

```
funciont()
// do something
function () {
  alert ("Hello")
}
```

### Macros and Registers

#### 录制宏

* 每个寄存器都是按a到z来定义的
* 在命令行模式下，输入q来触发宏录制，输入q来退出宏录制
* ```q<letter><commands>q```
* 如需执行多次，可以按如下格式输入 ```<number>@<letter>```

完整的宏看起来应该是这样的:

1. ```qd```	开始录制宏，记为寄存器 ```d```
2. ```..```	你可以在 vim 里进行各种操作
3. ```q```	按下q ，停止录制
4. ```@d```	执行你录制的宏 d
5. ```@@```	再次执行你刚刚录制的宏

* [Vim Macros 中文教程](http://my.oschina.net/linuxjd/blog/312847)
* [Vim Macros](http://vim.wikia.com/wiki/Macros)

### Advanced Movement

* H(ome), M(iddle), L(ast line)
* zt, zb, zz 把光标所在行移动到屏幕的顶部, 尾部, 或中间。
* m<letter> 可以标记当前位置，再次输入 '<letter> 则会跳转到这个记号位置，回到刚刚的位置用 ''

```
Ctrl + u, Ctrl + d, Ctrl + f, Ctrl + b, M, <number>H, <number>L, zt, zb, m<letter>, '<letter>, ''
```

### Buffers

```
:ls, :bnext, :bn, :bp, :b#, :bd<number>
```

### Using the Command Line Directly From Vim

* ft 是 file type 的意思
* 选中代码块之后也可执行命令, 比如 ```!coffee -c -s -p```

```
:! ls, :read !date, :set ft=javascript
:r !curl --silent https://github.com/jashkenas/coffeescript/blob/master/repl.js
```

## Vim as an Extension of Vi

### Windows and Tabs

```
:e <path and file>, :sp(lit) <path and file>, :vsp(lit) <path and file>, ctrl + w - w(h|j|k|l), ctrl + w - h, ctrl + w - l, ctrl + w + H(J|K|L), ctrl + w - <number> - (- | + | =), :sb<number of buffer>, :verb sb<number of buffer>, :tabedit <path and file>,  gt, gT,
:e app/apis/api.rb
```

* http://robots.thoughtbot.com/vim-splits-move-faster-and-more-naturally
* http://www.linux.com/learn/tutorials/442415-vim-tips-using-viewports

### Graphical Vim: GVim

### Indents and Folds

```
:set list, :set nolist, :set expandtab, :set shiftwidth=2, <number>>>, <number><<, :set smartindent, :set softtabstop=2
zf<number><hjkl>, zf%, zo, zc,
insert mode 下 Ctrl + t, Ctrl + d
```

## Customization

### The .vimrc File

```
set nocompatible

filetype on
filetype indent on
filetype plugin on

let mapleader = ","
syntax enable
"set foldmethod=syntax
set ignorecase
set hlsearch
set fileencoding=utf-8
set encoding=utf-8
set backspace=indent, eol, start
set ts=2 sts=2 expandtab

inoremap ...

set smartcase
set gdefault
set incsearch
set showmatch

set winwidth=84
set winheight=5
set winminheight=5
set winheight=999

set nolist
#set listchars=tab:>\, eol:
set guifont=PragmataPro
set guioptions=aAc
set guioptions-=Be
set number
set noswapfile
set visualbell
set cursorline
```

### Plugins and the Pathogen Tool

Manage Vim Plugins with Vundle

1. ```git clone https://github.com/gmarik/Vundle.vim.git bundle/Vundle.vim```
1. ```vim ~/.vimrc```
2. add the vundle configs:

```
set nocompatible               " be iMproved
filetype off                   " required!

set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" let Vundle manage Vundle
" required!
Bundle 'gmarik/vundle'

" Please note that that's how you add plugins
" My Bundles here:
"
" original repos on github
Bundle 'scrooloose/nerdtree'
" vim-scripts repos
Bundle 'FuzzyFinder'
" non github repos
Bundle 'git://git.wincent.com/command-t.git'

filetype plugin indent on " required!
```

3. open vim
4. ```:BundleInstall```
5. ```:NERDTree```

### Themes

### Mappings

### Final Tips


## 相关链接

* [Vimer的程序世界](http://www.vimer.cn/category/vim)
* [Vim应用大收集](http://be-evil.org/post-157.html)
