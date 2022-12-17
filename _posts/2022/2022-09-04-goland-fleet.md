---
layout: post
title:  "给 goland 配置成 fleet，插件、快捷键列表，调整内容"
date:   2022-09-04 08:10:00

categories: go
tags: editor
author: "Victor"
---

## 手工解锁新主题变身高仿 Fleet

1. 点击「随处搜索」按钮或者连按两次 `Shift` 键打开；
2. 输入 `Registry`；
3. 打开 `ide.experimental.ui` 相关的key以及 `debugger.new.tool.window.layout`; 可直接在列表中键入字符快速过滤列表；
4. 重启开发工具即可；

关闭新 UI，在步骤 3 点击 `恢复默认设置` 按钮。

## JVM选项

在大多数情况下，JVM选项的默认值应该是最佳的。这里是[文档](https://jetbrains.com.zh.xy2401.com/help/go/tuning-the-ide.html) 以下是最常修改的内容：

* -Xmx 改成 `-Xmx2048m`
* -Xms 改成 `-Xms1024m`

## 参考

* [IDEA插件一键解锁及手工解锁新主题变身高仿Fleet, VS Code](https://www.bilibili.com/video/BV1nW4y1Y7Ka?spm_id_from=333.999.0.0&vd_source=01880d66f14b1e64c622376868cbdec8)
