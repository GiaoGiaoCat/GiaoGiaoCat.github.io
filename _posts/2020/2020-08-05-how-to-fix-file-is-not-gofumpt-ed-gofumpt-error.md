---
layout: post
title:  "How to Fix some golangci-lint errors"
date:   2020-08-05 17:10:00

categories: go
tags: debug
author: "Victor"
---

## 1. How to Fix File is not `gofumpt`-ed (gofumpt) error

gofumpt is a stricter version of gofmt. The lint tool finds more errors and reports. One such error is that File is not gofumpt-ed. The solution to the problem is as follows:

Please install [gofumpt](https://github.com/mvdan/gofumpt)

The easiest solution is to run gofumpt on the troubled files.

`gofumpt -w <fileame>`

## 2. File is not `gci`-ed with -local github.com/golangci/golangci-lint

GCI, a tool that control golang package import order and make it always deterministic.

It handles empty lines more smartly than `goimport` does.

1. install `go get github.com/daixiang0/gci`
2. run `gci -w -local github.com/daixiang0/gci main.go`

## 3. File is not `goimports`-ed with -local github.com/golangci/golangci-lint

方案 1 手动执行格式化。

```shell
go get golang.org/x/tools/cmd/goimports

goimports -e -d -local github.com/golangci/golangci-lint internal/net/http/handler.go

goimports -w internal/net/http/handler.go
```

方案 2 配置 goland 的代码格式化规则。参考 [Running 'goimports' on save in GoLand](https://stackoverflow.com/questions/45590236/running-goimports-on-save-in-goland)

## 相关阅读

* [How to Fix File is not `gofumpt`-ed (gofumpt) error](https://freethreads.net/2020/07/28/how-to-fix-file-is-not-gofumpt-ed-gofumpt-error/)
* [add new linter to upport repairing import grouping/ordering #1257](https://github.com/golangci/golangci-lint/issues/1257)
