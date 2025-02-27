---
layout: post
title:  "如何通过命令行修改 macOS 系统下某一个文件类型的默认启动应用"
date:   2020-11-17 17:10:00

categories: tool
tags: terminal mac
author: "Victor"
---

## duti

安装一下 `brew install duti`。

### Get the default app id for .sh files:

可以查看 `.sh` 文件默认使用哪个应用启动。

```bash
duti -x sh

output:
  Brackets.app
  /opt/homebrew-cask/Caskroom/brackets/1.6/Brackets.app
  io.brackets.appshell # 这里是 Brackets 的 app id
```

### Use the app id for all .md files:

假设我希望使用 Typora 打开所有 `.md` 类型的文件。

```bash
osascript -e 'id of app "Typora"'
duti -s abnerworks.Typora .md all
```

或者一步到位：

```bash
duti -s $(osascript -e 'id of app "Typora"') .md all
```

## 相关阅读

* [How to change default app for all files of particular file type through terminal in OS X?](https://superuser.com/questions/273756/how-to-change-default-app-for-all-files-of-particular-file-type-through-terminal)
