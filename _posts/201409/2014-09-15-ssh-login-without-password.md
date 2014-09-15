---
layout: post
title:  "不输密码登录 SSH"
date:   2014-09-15 17:10:00
categories: tool
tags: terminal server
author: "Victor"
---

第一步：安装好 ssh

```bash
apt-get install ssh
```

第二步：配置 openSSH 为允许 PubkeyAuthentication 认证

```bash
vim /etc/ssh/sshd_config
PubkeyAuthentication yes # 去掉注释
AuthorizedKeysFile ~/.ssh/authorized_keys # 去掉注释
useDNS no # 不改的话 ssh 客户端连接的时候会很慢，这一步可以忽略
```
第三步：把公钥加入到 authorized_keys 中，以让服务端认识你的公钥

第四步：重启 sshd 服务

```bash
service ssh restart
```

第五步：增强安全性

为了安全起见，如果你的主机只有你一两个人需要登录主机，可以禁用掉其它的 Passord 认证方式。

```bash
vim /etc/ssh/sshd_config
PasswordAuthentication no # 去掉注释, yes 改成 no
ChallengeResponseAuthentication no #  去掉注释, yes 改成 no
```
