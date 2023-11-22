---
layout: post
title:  "修复 goland 更新后就各种报错"
date:   2023-11-21 08:10:00

categories: go
tags: editor
author: "Victor"
---

每次更新 goland 之后都会提示各种语法错误，找不到依赖库之类的。

File -> Invalidate Caches 全选之后重启，可以修复这个问题。

以上操作建议在有软路由的环境或者切换到手机网络执行，纯粹是因为公司的办公网络翻墙不稳定，所以有时候更新库都会失败。
