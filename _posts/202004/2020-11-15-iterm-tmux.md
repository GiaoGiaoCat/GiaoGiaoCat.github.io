---
layout: post
title:  "tmux"
date:   2020-11-14 17:10:00

categories: tool
tags: terminal
author: "Victor"
---

## tmux 是什么

直接看相关阅读中 [Tmux 使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html) 就好。

tmux 是把多个虚拟窗口看成一个物理窗口(terminal multiplexer)，通过 tmux 的命令来操作这些窗口集合，方便切换且不会打乱你原来的工作环境，有点类似于 Screen - GNU，两者强大之处在于能把整个窗口都切换到后台运行，需要的时候再切换回来， 但是 tmux 的可操控性更好。

screen 只支持窗口(windows)模式，tmux 支持多 panes 模式(与item2一致)，一个 window 里面可以分割为多个 panes。

## 简单使用

输入 tmux 启动，底部有一个状态栏。状态栏的左侧是窗口信息（编号和名称），右侧是系统信息。

前缀键 `Ctrl+b`，然后再按命令键。比如 ? 显示帮助信息。

```bash
tmux new -s <session-name> // 新建会话
tmux ls // 查看当前所有的 tmux 会话
tmux attach -t 0 // 使用会话编号接入会话
tmux attach -t <session-name> // 使用会话名称接入会话
tmux kill-session -t 0 或 <session-name> // 杀死会话
tmux switch -t 0 或 <session-name> // 切换会话
tmux rename-session -t 0 或 <session-name> // 重命名会话

Ctrl+b d // 分离会话
Ctrl+b s // 列出所有会话
Ctrl+b $ // 重命名当前会话

Ctrl+b % // 划分左右两个窗格
Ctrl+b " // 划分上下两个窗格"
Ctrl+b <arrow key> // 光标切换到其他窗格
Ctrl+b x // 关闭当前窗格
Ctrl+b ! // 将当前窗格拆分为一个独立窗口
Ctrl+b z // 当前窗格全屏显示，再使用一次会变回原来大小
Ctrl+b Ctrl+<arrow key> // 按箭头方向调整窗格大小
```

## 进阶玩法

### 安装插件管理器 TPM

抄自 [Tmux Plugin Manager使用及具体插件](https://www.cnblogs.com/hongdada/p/13528984.html)。

```bash
# 把管理器文件安装到`~/.tmux/plugins/tpm`之下 此前这些目录是不存在的
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

新建配置文件 `vim ~/.tmux.conf`

将下面内容复制到`~/.tmux.conf`

```bash
# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
# Other examples:
# set -g @plugin 'github_username/plugin_name'
# set -g @plugin 'git@github.com/user/plugin'
# set -g @plugin 'git@bitbucket.com/user/plugin'
# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
```

使其生效 `tmux source-file ~/.tmux.conf`

快捷键管理插件：

1. 添加新的插件 `~/.tmux.conf` 与 `set -g @plugin '...'`
2. 按 prefix + I（来获取插件

之后就会把插件已克隆到 `~/.tmux/plugins/dir`。

卸载插件：

1. 从列表中删除（或注释掉）插件
2. 按 prefix + alt + u 删除插件

更新插件：

1. prefix + U

也可以在此处找到插件目录并将其删除。

## 推荐插件

### 复制粘贴插件

安装完成后默认使用鼠标选中后，松开鼠标右键即为复制，iTerm2 提供该功能所以我没装。

```bash
set -g @plugin 'tmux-plugins/tmux-yank'
```

### tmux-resurrect

tmux 永久保存插件 - 手动

```bash
set -g @plugin 'tmux-plugins/tmux-resurrect'
```

要保存 Tmux 会话 ， 我们只要按 `前缀键 + Ctrl-s` 就可以了 。 此时 Tmux 状态栏会显示 “Saving ...” 字样，完毕后会提示 Tmux 环境已保存。

Tmux Resurrect 会将 Tmux 会话的详细信息以文本文件形式保存到 `~/.tmux/resurrect` 目录。

还原则按 `前缀键 + Ctrl-r` 即可。

默认情况下，仅还原保守的程序列表 `vi vim nvim emacs man less more tail top htop irssi weechat mutt`。

我们可能还需要保存当前每个窗格运行的程序。类似 `vim, less, man` 这些程序 tmux-resurrect 会自动恢复，其他的则需要配置：

```bash
set -g @resurrect-processes 'ssh mysql redis-server npm'
```

这个插件可以保存和恢复 tmux 窗格的内容，可以通过添加以下行来启用此功能 .tmux.conf：

```bash
set -g @resurrect-capture-pane-contents 'on'
```

### tmux-continuum

tmux 永久保存插件 - 自动

* continuous saving of tmux environment
* automatic tmux start when computer/server is turned on
* automatic restore when tmux is started

### tmux-power

让状态栏好看。

```bash
set -g @plugin 'wfxr/tmux-power'
set -g @tmux_power_theme 'gold' # 调整颜色
set -g @tmux_power_session_icon '🔑';
set -g @tmux_power_user_icon '🙂';
set -g @tmux_power_time_icon '🕒';
set -g @tmux_power_date_icon '📆';
```

颜色可以调整 redwine、moon、forest、violet、snow、coral、sky 等顏色。你甚至可以直接輸入色碼（如 #FF4500），設定成你想要的顏色。

### tmux-prefix-highlight

显示自己是否按了 tmux-prefix 键

```bash
set -g @plugin 'tmux-plugins/tmux-prefix-highlight'
set -g @tmux_power_prefix_highlight_pos 'L' # 和 tmux-power 相容
```

### tmux-plugins/tmux-open

Plugin for opening highlighted selection directly from Tmux copy mode.

* o - "open" a highlighted selection with the system default program. open for OS X or xdg-open for Linux.
* Ctrl-o - open a highlighted selection with the $EDITOR
* Shift-s - search the highlighted selection directly inside a search engine (defaults to google).

用这个插件要先学会 tmux 的 copy 模式。

1. 在 ~/.tmux.conf 中启用 vi 模式
2. `PREFIX [` 进入复制模式
3. 按 space 开始复制，移动光标选择复制区域
4. 按 Enter 复制并退出 copy-mode
5. 将光标移动到指定位置，按 `PREIFX ]` 粘贴
6. 可以用 man tmux 查看更详细的说明

```bash
The following commands are supported in copy mode:

Command                                      vi              emacs
append-selection
append-selection-and-cancel                  A
back-to-indentation                          ^               M-m
begin-selection                              Space           C-Space
bottom-line                                  L
cancel                                       q               Escape
clear-selection                              Escape          C-g
copy-end-of-line [<prefix>]                  D               C-k
copy-line [<prefix>]
copy-pipe <command> [<prefix>]
copy-pipe-no-clear <command> [<prefix>]
copy-pipe-and-cancel <command> [<prefix>]
copy-selection [<prefix>]
copy-selection-no-clear [<prefix>]
copy-selection-and-cancel [<prefix>]         Enter           M-w
cursor-down                                  j               Down
cursor-down-and-cancel
cursor-left                                  h               Left
cursor-right                                 l               Right
cursor-up                                    k               Up
end-of-line                                  $               C-e
goto-line <line>                             :               g
halfpage-down                                C-d             M-Down
halfpage-down-and-cancel
halfpage-up                                  C-u             M-Up
history-bottom                               G               M->
history-top                                  g               M-<
jump-again                                   ;               ;
jump-backward <to>                           F               F
jump-forward <to>                            f               f
jump-reverse                                 ,               ,
jump-to-backward <to>                        T
jump-to-forward <to>                         t
middle-line                                  M               M-r
next-matching-bracket                        %               M-C-f
next-paragraph                               }               M-}
next-space                                   W
next-space-end                               E
next-word                                    w
next-word-end                                e               M-f
other-end                                    o
page-down                                    C-f             PageDown
page-down-and-cancel
page-up                                      C-b             PageUp
previous-matching-bracket                                    M-C-b
previous-paragraph                           {               M-{
previous-space                               B
previous-word                                b               M-b
rectangle-toggle                             v               R
scroll-down                                  C-e             C-Down
scroll-down-and-cancel
scroll-up                                    C-y             C-Up
search-again                                 n               n
search-backward <for>                        ?
search-backward-incremental <for>                            C-r
search-backward-text <for>
search-forward <for>                         /
search-forward-incremental <for>                             C-s
search-forward-text <for>
search-reverse                               N               N
select-line                                  V
select-word
start-of-line                                0               C-a
stop-selection
top-line                                     H               M-R
```

### tmux-sidebar

it opens a tree directory listing for the current path.

* prefix + Tab - toggle sidebar with a directory tree
* prefix + Backspace - toggle sidebar and move cursor to it (focus it)

### tmux-jump

* tmux-prefix + j and enter the first character of a word.
* The screen will rerender and highlight the keys to press to jump to the word.
* Type the key sequence of the word to jump to.
* The cursor moves to the word.

## 相关阅读

* [Iterm2整合Tmux利器](https://tried.cc/Iterm2TmuxIntegration/)
* [Mac下iTerm2＋Tmux配置](https://segmentfault.com/a/1190000003001555)
* [Tmux 使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)
* [tmux Integration Best Practices](https://gitlab.com/gnachman/iterm2/-/wikis/tmux-Integration-Best-Practices)
* [Tmux Cheat Sheet & Quick Reference](https://tmuxcheatsheet.com) 建议收藏经常看看
* [3 個 tmux 常用的套件](https://noob.tw/tmux-plugins/)
* [Vi mode in tmux](https://sanctum.geek.nz/arabesque/vi-mode-in-tmux/)
* [Awesome Tmux](https://github.com/rothgar/awesome-tmux)
