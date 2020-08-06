---
layout: post
title:  "How to Fix File is not `gofumpt`-ed (gofumpt) error"
date:   2020-08-05 17:10:00

categories: go
tags: debug
author: "Victor"
---

gofumpt is a stricter version of gofmt. The lint tool finds more errors and reports. One such error is that File is not gofumpt-ed. The solution to the problem is as follows:

Please install [gofumpt](https://github.com/mvdan/gofumpt)

The easiest solution is to run gofumpt on the troubled files.

`gofumpt -w <fileame>`

## 相关阅读

* [How to Fix File is not `gofumpt`-ed (gofumpt) error](https://freethreads.net/2020/07/28/how-to-fix-file-is-not-gofumpt-ed-gofumpt-error/)
