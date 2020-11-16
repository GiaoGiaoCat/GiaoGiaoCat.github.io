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

## 命令列表

```bash
Ctrl+b // 激活控制台；此时以下按键生效
系统操作
? // 列出所有快捷键；按q返回
d // 脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话
D // 选择要脱离的会话；在同时开启了多个会话时使用
Ctrl+z // 挂起当前会话
r // 强制重绘未脱离的会话
s // 选择并切换会话；在同时开启了多个会话时使用
: // 进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器
[ // 进入复制模式；此时的操作与vi/emacs相同，按q/Esc退出
~ // 列出提示信息缓存；其中包含了之前tmux返回的各种提示信息
窗口操作
c // 创建新窗口
& // 关闭当前窗口
数字键 // 切换至指定窗口
p // 切换至上一窗口
n // 切换至下一窗口
l // 在前后两个窗口间互相切换
w // 通过窗口列表切换窗口
, // 重命名当前窗口；这样便于识别
. // 修改当前窗口编号；相当于窗口重新排序
f // 在所有窗口中查找指定文本
面板操作
” // 将当前面板平分为上下两块
% // 将当前面板平分为左右两块
x // 关闭当前面板
! // 将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板
Ctrl+方向键 // 以1个单元格为单位移动边缘以调整当前面板大小
Alt+方向键 // 以5个单元格为单位移动边缘以调整当前面板大小
Space // 在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled
q // 显示面板编号
o // 在当前窗口中选择下一面板
方向键 // 移动光标以选择面板
{ // 向前置换当前面板
} // 向后置换当前面板
Alt+o // 逆时针旋转当前窗口的面板
Ctrl+o // 顺时针旋转当前窗口的面板
```

## 相关阅读

* [Iterm2整合Tmux利器](https://tried.cc/Iterm2TmuxIntegration/)
* [Mac下iTerm2＋Tmux配置](https://segmentfault.com/a/1190000003001555)
* [Tmux 使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)
* [tmux Integration Best Practices](https://gitlab.com/gnachman/iterm2/-/wikis/tmux-Integration-Best-Practices)
