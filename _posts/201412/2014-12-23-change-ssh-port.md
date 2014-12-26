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

-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
```

执行 ```/etc/init.d/iptables restart``` 或 ```service iptables restart```。

重启防火墙

**重启后生效**

* 开启: ```chkconfig iptables on```
* 关闭: ``` chkconfig iptables off```

**即时生效，重启后失效**

* 开启: ```service iptables start```
* 关闭: ```service iptables stop```

## 限制 SSH 登陆的 IP

```bash
vim /etc/hosts.allow

sshd:192.168.0.241 # 限制只有 192.168.0.241 的 IP 可以访问
```

## 防火墙脚本

```bash
#允许所有IP对本机80端口的访问
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p udp --dport 80 -j ACCEPT
#提高本地数据包的优先权：放在第1位
iptables -t mangle -A OUTPUT -p tcp -m tcp --dport 22 -j MARK –set-mark 1
iptables -t mangle -A OUTPUT -p tcp -m tcp --dport 22 -j RETURN
iptables -t mangle -A OUTPUT -p icmp -j MARK –set-mark 1
iptables -t mangle -A OUTPUT -p icmp -j RETURN
#提高tcp初始连接(也就是带有SYN的数据包)的优先权是非常明智的：
iptables -t mangle -A PREROUTING -p tcp -m tcp –tcp-flags SYN,RST,ACK SYN -j MARK –set-mark 1
iptables -t mangle -A PREROUTING -p tcp -m tcp –tcp-flags SYN,RST,ACK SYN -j RETURN
#提高ssh数据包的优先权：放在第1类，要知道ssh是交互式的和重要的，不容待慢哦
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 22 -j MARK –set-mark 1
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 22 -j RETURN
#封锁55.55.55.55对本机所有端口的访问，暂不使用，我的注释掉了。
iptables -I INPUT -s 55.55.55.55 -j DROP
#禁止55.55.55.55对80端口的访问
iptables -I INPUT -s 55.55.55.55 -p TCP --dport 80 -j DROP
```

## 参考

* [Linux的iptables常用配置范例](http://www.ha97.com/3928.html)
* [Ubuntu Documentation - Firewall](https://help.ubuntu.com/10.04/serverguide/firewall.html)
* [How to Manually Edit ufw Rules on Ubuntu Linux](https://scottlinux.com/2012/08/25/how-to-manually-edit-ufw-rules-on-ubuntu-linux/)
* [Using ufw / iptables in Ubuntu 8.04 LTS](https://pario.no/2008/05/21/using-ufw-iptables-in-ubuntu-804-lts/)
