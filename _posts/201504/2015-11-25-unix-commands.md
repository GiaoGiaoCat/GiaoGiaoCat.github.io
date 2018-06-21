---
layout: post
title:  "Unix/Linux 命令技巧"
date:   2015-11-25 14:00:00
categories: tool
tags: terminal
author: "Victor"
---

### 删除一个大文件

我在生产服务器上有一个很大的200GB的日志文件需要删除。我的 `rm` 和 `ls` 命令已经崩溃，我担心这是由于巨大的磁盘IO造成的，要删除这个大文件，输入：

```bash
> /path/to/file.log

# 或使用如下格式
: > /path/to/file.log
# 然后删除它
rm /path/to/file.log
```

### 还原被删除的 /tmp 文件夹

我意外地删除了 `/tmp` 文件夹。要还原它，我需要这么做：

```bash
mkdir /tmp
chmod 1777 /tmp
chown root:root /tmp
ls -ld /tmp
```

### 锁定一个文件夹

为了我的数据隐私，我想要锁定我文件服务器下的 `/downloads` 文件夹。因此我运行了：

```bash
chmod 0000 /downloads

# root用户仍旧可以访问，而ls和cd命令则不工作。要还原它用：

chmod 0755 /downloads
```

### 易读格式

传递 `-h` 或者 `-H`（和其他选项）选项给GNU或者BSD工具来获取像 `ls、df、du` 等命令以易读的格式输出：

```bash
ls -lh
# 以易读的格式 (比如： 1K 234M 2G)
df -h
df -k
# 以字节、KB、MB 或 GB 输出：
free -b
free -k
free -m
free -g
# 以易读的格式输出 (比如 1K 234M 2G)
du -h
# 以易读的格式显示文件系统权限
stat -c %A /boot
# 比较易读的数字
sort -h -a file
# 在Linux上以易读的形式显示cpu信息
lscpu
lscpu -e
lscpu -e=cpu,node
# 以易读的形式显示每个文件的大小
tree -h
tree -h /boot
```

### 在Linux系统中显示已知的用户信息

只要输入：

```bash
## linux 版本 ##
lslogins
## BSD 版本 ##
logins
```

示例输出：

```bash
UID USER      PWD-LOCK PWD-DENY LAST-LOGIN GECOS
  0 root             0        0   22:37:59 root
  1 bin              0        1            bin
  2 daemon           0        1            daemon
  3 adm              0        1            adm
  4 lp               0        1            lp
  5 sync             0        1            sync
  6 shutdown         0        1 2014-Dec17 shutdown
  7 halt             0        1            halt
  8 mail             0        1            mail
 10 uucp             0        1            uucp
 11 operator         0        1            operator
 12 games            0        1            games
 13 gopher           0        1            gopher
 14 ftp              0        1            FTP User
 27 mysql            0        1            MySQL Server
 38 ntp              0        1
 48 apache           0        1            Apache
 68 haldaemon        0        1            HAL daemon
 69 vcsa             0        1            virtual console memory owner
 72 tcpdump          0        1
 74 sshd             0        1            Privilege-separated SSH
 81 dbus             0        1            System message bus
 89 postfix          0        1
 99 nobody           0        1            Nobody
173 abrt             0        1
497 vnstat           0        1            vnStat user
498 nginx            0        1            nginx user
499 saslauth         0        1            "Saslauthd user"
```

### 我如何删除意外在当前文件夹下解压的文件？

我意外在 `/var/www/html/` 下解压了一个 `tarball`。它搞乱了 `/var/www/html` 下的文件，你甚至不知道哪些是误解压出来的。最简单修复这个问题的方法是：

```bash
cd /var/www/html/
/bin/rm -f "$(tar ztf /path/to/file.tar.gz)"
```

### 将文件复制到多个目录中

不必运行：

```bash
cp /path/to/file /usr/dir1
cp /path/to/file /var/dir2
cp /path/to/file /nas/dir3
```

运行下面的命令来复制文件到多个目录中：

```bash
echo /usr/dir1 /var/dir2 /nas/dir3 |  xargs -n 1 cp -v /path/to/file
```

### 快速找出两个目录的不同

`diff` 命令会按行比较文件。但是它也可以比较两个目录：

```bash
ls -l /tmp/r
ls -l /tmp/s
# 使用 diff 比较两个文件夹
diff /tmp/r/ /tmp/s/
```

### 可以看见输出并将其写入到一个文件中

如下使用 `tee` 命令在屏幕上看见输出并同样写入到日志文件 my.log 中：

```bash
mycoolapp arg1 arg2 input.file | tee my.log
```

`tee` 可以保证你同时在屏幕上看到 mycoolapp 的输出并写入文件 my.log。


## 更多阅读

* [20个 Unix/Linux 命令技巧](https://community.emc.com/message/873900)
