---
layout: post
title:  "Perfect Workflow in Sublime Text 2"
date:   2014-08-28 10:33:00
categories: tool
tags: editor
author: "Victor"
---

## Getting Started

### Base Settings

你可以在```Setting - User```中，设置自己喜欢的字体，Theme等。

### Services and Opening Sublime From the Terminal

#### 给文件夹的Services选项，添加从sublime text2打开的功能

1. Open Automator，Choose Service
2. Actions Name is Run Shell Script
3. Service receives selected is files or folders
4. shell is /bin/bash
5. Pass input: as arguments
6. arguments is ``` /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl -n "$@" ```
7. Save it as named Open-in-Sublime

#### How to Remove Services from the Contextual Menu in Mac OS X

Here is an [article](http://osxdaily.com/2013/05/14/remove-services-contextual-menu-mac-os-x/).

#### Open file or folder by sublime text 2 in terminal

1. Open terminal.
2. run ``` ln -s /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl /usr/local/bin/sublime ```
3. If u get *Permission denied*, then u need to run ->> ``` sudo !! ```
4. ```!!``` will be run the lasted command.
5. Remember to restart ur terminal.
6. Right now u can run sublime text 2 by ```sublime``` command in your terminal. If it does't work. take a look at [this page](https://gist.github.com/olivierlacan/1195304).

## Perfect Workflow in Sublime Text 2: Useful Shortcut Key

### Column Selection

* ```Ctrl+Shift+Up``` or ```Ctrl+Shift+Down```
* ```Option+Mouse```
* ```Cmd+Shift+L``` Select a block of lines, and then split it into many selections, one per line.

### Add Multiple Cursors and Incremental Search

* ```Cmd+Opt+F``` Replace in current file.
* ```Cmd+I``` Incremental Find.
* ```Cmd+L``` Expand selection to Line.
* ```Cmd+D``` Expand selection to Word. ```Cmd+U``` Undo Selection.
* ```Cmd+Ctrl+G``` Selection all of the word in current file.
* ```Ctrl+Shift+K``` Delete Line.
* ```Ctrl+K``` Delete to End.
* ```Cmd+Del``` Delete to Beginning.

### The Command Palette

* ```Cmd+Shift+p``` can open command palette. With a simple keystroke, you can make any number of modifications in seconds. Example: *Set Syntax*, *Toggle Minimap* and so on.
* ```Cmd+K, Cmd+B``` toggle sidebar.

### Code Folding
* ```Cmd+k+1``` code folding.
* ```Cmd+k+j``` unfold all blocks.

### Instant File Changing

```Cmd+P``` You’ll find that Sublime’s instant file switching capabilities is quicker than any other editor you’ve used. Type an ```@``` character, and start browsing by symbol. Type ```#``` to search within the file, or ```:``` to go to a line number. Combining these together, for example, ```tp@rf``` may take you to a function called *read_file* within a file named *text_parser.py*. Similarly, ```tp:100``` would take you to line 100 of the same file.

### Symbols

```Cmd+R``` Whether in a class or CSS file, Sublime allows us to view and transition to all symbols. There is a other way to open symbols palette. ```Cmd+P ``` then type ```@```.

### Instant File Changing and Goto Symbols
```Cmd+P``` then type filename@symbols, example *style@body*

### Key Bindings

You can change shortcut key in *Preferences - Key Bindings - User*.

### Installing Plugins Without Package Control

It's really easy to do, just copy files into Package folder. *Preferences - Browse Packages...*.

### Package Control

[Sublime Package Control](http://wbond.net/sublime_packages/package_control) is a full-featured package manager that helps discovering, installing, updating and removing packages for Sublime Text 2. It features an automatic upgrader and supports GitHub, BitBucket and a full channel/repository system.

## Snippets and Essential Plugins

### Snippets

There are two videos [Your First Snippet](https://tutsplus.com/lesson/your-first-snippet/) and [Adding Snippets Through Package Control](https://tutsplus.com/lesson/adding-snippets-through-package-control/).

#### 创建一个快捷键

* 在 /Users/[Your Name]/Library/Application Support/Sublime Text 2/Packages/User 下面创建一个文件夹
* 文件夹命名规则，如果你期望快捷键在 HTML 格式下可用，就把文件夹命名为HTML
* 创建一个文件类似 custom.sublime-snippet 其中内容如下

```xml
<snippet>
  <content><![CDATA[<%${1:=} ${2:} %>]]>
  </content>
  <tabTrigger>re</tabTrigger>
</snippet>
```

### Essential Plugins

* **Zen Coding** is a plugin for HTML developer.
* [Emmet](https://tutsplus.com/lesson/emmet/) is a better plugin than **Zen Coding** for HTML and CSS(NOT for SASS) developer.
* [Sublime Prefixr](https://github.com/wbond/sublime_prefixr) plugin runs CSS through the [Prefixr](http://prefixr.com/) API for Sublime Text 2. It could be instantly add (or remove) the necessary CSS3 prefixes for your stylesheets. Please see [http://wbond.net/sublime_packages/prefixr](http://wbond.net/sublime_packages/prefixr) for install instructions, screenshots and documentation.
* [Nettuts+](https://tutsplus.com/lesson/fetch-files-with-ease/) gives you an easy way to instantly pull in new assets into your projects.
* [AdvancedNewFile](https://tutsplus.com/lesson/lightning-fast-folder-and-file-creation/) able to create files so quickly in Sublime.
* [Sidebar Enhancements](https://tutsplus.com/lesson/sidebar-enhancements/) make many options are provided, if you right-click on a file within the sidebar.
* [Sublime Linter](https://tutsplus.com/lesson/sublime-linter/) is the first tester in your arsenal.
* **Sexy Code Snippet Management With Gists**
* **DocBlockr**
* **Pretty Task Management**
* [HTTP Requests Within Sublime](https://tutsplus.com/lesson/http-requests-within-sublime/)
* **LiveReload**

### Tips, Techniques and Modifications
* Regular Expressions in Sublime
* Vintage Mode
* Quicker Stylesheet References
* Joining Lines, just use ```Cmd+J```
* Sublime and Markdown with Marked
* [All About Projects](https://tutsplus.com/lesson/all-about-projects/)
* [Configuring and Mastering Split Windows](https://tutsplus.com/lesson/configuring-and-mastering-split-windows/)
* Custom Builds
