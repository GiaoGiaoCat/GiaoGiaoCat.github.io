---
layout: post
title:  "利用 Mosh 改善 SSH"
date:   2015-08-27 14:00:00
categories: tool
tags: terminal
author: "Victor"
---

在高延迟的网络下，Mosh 比 SSH 流畅多了，而且3G网络下，丢失网络变更IP后，终端连接不会中断。

## 安装

`brew install mobile-shell`

注: Server, Client 都需要安裝 mosh 才可以。

## 用法

经典用法

Mosh will log the user in via SSH, then start a connection on a UDP port between 60000 and 61000.

`$ mosh chewbacca.norad.mil`

Different username

`$ mosh potus@ackbar.bls.gov`

Server binary outside path

`$ mosh --server=/tmp/mosh-server r2d2`

Selecting Mosh UDP port

`$ mosh -p 1234 darth`

Selecting SSH port

`$ mosh --ssh="ssh -p 2222" figrindan`

## 链接

* [Mosh 官方](https://mosh.mit.edu/)
