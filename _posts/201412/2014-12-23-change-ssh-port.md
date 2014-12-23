---
layout: post
title:  "修改 SSH 默认 22 端口的方法"
date:   2014-12-23 17:50:00
categories: tool
tags: terminal server
author: "Victor"
---

## 改 Linux SSH 的默认端口 22

```bash
vim /etc/ssh/sshd_config

Port 22 # change to other number.
```

然后执行 ```/etc/init.d/sshd restart```。

然后编辑防火墙，启用刚刚设置的端口。

```bash
vi /etc/sysconfig/iptables
```

执行 ```/etc/init.d/iptables restart```。

## 限制 SSH 登陆的 IP

```bash
vim /etc/hosts.allow

sshd:192.168.0.241 # 限制只有 192.168.0.241 的 IP 可以访问
```
