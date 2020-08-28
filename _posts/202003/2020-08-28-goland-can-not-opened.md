---
layout: post
title:  "Mac 安装 Goland2020.1 无法启动"
date:   2020-08-28 17:10:00

categories: tool
tags: editor
author: "Victor"
---

首先查看启动的错误日志，在 terminal 中执行以下命令：

```bash
/Applications/GoLand.app/Contents/MacOS/goland
```

显示信息如下：

```bash
➜  ~ /Applications/GoLand.app/Contents/MacOS/goland ; exit;
2020-04-10 10:29:57.895 goland[1523:31631] allVms required 1.8*,1.8+
2020-04-10 10:29:57.897 goland[1523:31634] Current Directory: /Users/Donne
2020-04-10 10:29:57.897 goland[1523:31634] Value of GOLAND_VM_OPTIONS is (null)
2020-04-10 10:29:57.897 goland[1523:31634] Processing VMOptions file at /Users/Donne/Library/Application Support/JetBrains/GoLand2020.1/goland.vmoptions
2020-04-10 10:29:57.897 goland[1523:31634] Done
Error: could not find libjava.dylib
Failed to GetJREPath()
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
Error opening zip file or JAR manifest missing : /Applications/GoLand.app/Contents/bin/jetbrains-agent.jar
Error occurred during initialization of VM
agent library failed to init: instrument
```

原因：因为之前用过盗版软件，所以 goland.vmoptions 最后一行添加过 jetbrains-agent.jar 路径。
解决方法：删除了 /Users/Donne/Library/Application Support/JetBrains/GoLand2020.1/goland.vmoptions 文件最后一行的 jetbrains-agent.jar 路径。

## 原文

* [Mac 安装 Goland2020.1 无法启动](https://www.it610.com/article/1296715926288277504.htm)
