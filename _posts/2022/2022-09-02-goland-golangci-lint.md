---
layout: post
title:  "给 goland 配置 golangci-lint"
date:   2022-09-02 08:10:00

categories: go
tags: editor
author: "Victor"
---

## 问题

每次启动 goland 的时候，右下角总是提示我 golangci-lint 该升级了。什么玩意，我一天更新 brew 有 800 次啊。

## 解决

我是用 brew 安装的，所以先确认自己已经升级到最新版。

```bash
brew upgrade golangci/tap/golangci-lint
```

然后在看看当前的版本，没错就是最新版。

```bash
golangci-lint --version
```

看看系统默认的 golangci-lint 装在哪，再看看 brew 的装载哪。

```bash
# 系统
where golangci-lint

/opt/homebrew/bin/golangci-lint
/usr/local/bin/golangci-lint
/opt/homebrew/bin/golangci-lint
/usr/local/bin/golangci-lint

# brew
brew list golangci-lint

/opt/homebrew/Cellar/golangci-lint/1.49.0/bin/golangci-lint
/opt/homebrew/Cellar/golangci-lint/1.49.0/etc/bash_completion.d/golangci-lint
/opt/homebrew/Cellar/golangci-lint/1.49.0/share/fish/vendor_completions.d/golangci-lint.fish
/opt/homebrew/Cellar/golangci-lint/1.49.0/share/zsh/site-functions/_golangci-lint
```

然后看看 goland 用的是哪个目录的，打开配置在 Tools -> Go linter。擦，果然用错了。

把目录改成 `/opt/homebrew/Cellar/golangci-lint/1.49.0/bin/golangci-lint`，如果嫌手动选目录麻烦，跳转到指定目录快捷键 command+shift+G。

保存，重启，完美。
