---
layout: post
title:  "Linux系统最大文件打开数优化，解决Too many open files报错"
date:   2020-11-03 17:10:00

categories: tool
tags: terminal
author: "Victor"
---

这是一个 Linux 系统常见的故障，

## 无脑解决

### 临时解决

临时的解决办法很简单，先执行 `ulimit -n 65535`，然后 `ulimit -n` 查看当前的最大文件打开数是否已优化。

```bash
# 通过 ulimit -Sn 设置最大打开文件描述符数的 soft limit，注意 soft limit 必须小于 hard limit
ulimit -Sn 160000

# 同时设置 soft limit 和 hard limit。对于非 root 用户只能设置比原来小的hard limit。
ulimit -n 180000
```

### 永久生效

使用方案 1，登陆普通账号的时候，会报无权限错误，所以推荐使用方案 2。

不管使用哪种方法，修改生效后，需要重启应用才行，否则程序将延续使用旧的环境设置。

#### 方案 1

`ulimit -n 65535` 添加到 `/etc/profile` 的最后即可。

```bash
echo "ulimit -n 65535" >>/etc/profile
#刷新配置
source /etc/profile
```

#### 方案 2

root 权限下，修改 `/etc/security/limits.conf` 文件，在最后添加以下内容即可，这是 百度 的推荐：

```bash
* soft nofile 65535
* hard nofile 65535
root soft nofile 65535
root hard nofile 65535
```

**设置 nofile 的 hard limit 还有一点要注意的就是 hard limit 不能大于 /proc/sys/fs/nr_open，假如 hard limit 大于 nr_open，注销后将无法正常登录。**

## `/etc/security/limits.conf` 配置文件说明

linux 资源限制配置文件是 `/etc/security/limits.conf`，限制用户进程的数量系统的稳定性非常重要。该文件限制着用户可以使用的最大文件数，最大线程，最大内存等资源使用量。

```bash
* soft nofile 655350  #任何用户可以打开的最大的文件描述符数量，默认1024，这里的数值会限制tcp连接
* hard nofile 655350
* soft nproc  655350  #任何用户可以打开的最大进程数
* hard nproc  650000

@student hard nofile 65535
@student soft nofile 4096
@student hard nproc 50  #学生组中的任何人不能拥有超过50个进程，并且会在拥有30个进程时发出警告
@student soft nproc 30
```

* soft: 代表警告的设定，可以超过这个设定值，但是超过后会有警告
* hard: 代表严格的设定，是一个真正意义的阀值，超过就会报错
* nproc: 是操作系统级别对每个用户创建的进程数的限制
* nofile: 是每个进程可以打开的文件数的限制
* 第一列表示用户，* 表示所有用户

```bash
soft nproc # 单个用户可用的最大进程数量(超过会警告);
hard nproc # 单个用户可用的最大进程数量(超过会报错);
soft nofile # 可打开的文件描述符的最大数(超过会警告);
hard nofile # 可打开的文件描述符的最大数(超过会报错);
```

需要重启之后才能生效，`reboot`。

### 注意事项

1. 一般 soft 的值会比 hard 小，也可相等。
2. `/etc/security/limits.d/` 里面配置会覆盖 `/etc/security/limits.conf` 的配置
3. 只有 root 用户才有权限修改 `/etc/security/limits.conf`
4. 如果 limits.conf 没有做设定，则默认值是 1024

## 一些有用的命令

```bash
# 所有用户创建的进程数
ps h -Led -o user | sort | uniq -c | sort -n

# 系统最大打开文件描述符数
cat /proc/sys/fs/file-max

# 设置
vim /etc/sysctl.conf
fs.file-max = 6553600

# 进程最大打开文件描述符数
# 默认查看的是 soft limit
ulimit -n

# 查看 hard limit
ulimit -Hn

# 查看当前系统使用的打开文件描述符数
# 其中第一个数表示当前系统已分配使用的打开文件描述符数，第二个数为分配后已释放的（目前已不再使用），第三个数等于file-max。
cat /proc/sys/fs/file-nr
```

### ulimit -a/n/H/S 都有什么含义

* ulimit -a 显示当前所有的资源限制
* ulimit -H 设置硬件资源限制
* ulimit -S 设置软件资源限制
* ulimit -n 设置进程最大打开文件描述符数

### 注意

* 所有进程打开的文件描述符数不能超过 /proc/sys/fs/file-max
* 单个进程打开的文件描述符数不能超过 user limit中nofile 的 soft limit
* nofile 的 soft limit 不能超过其 hard limit
* nofile 的 hard limit 不能超过 /proc/sys/fs/nr_open

## 相关阅读

* [Linux系统最大文件打开数优化，解决Too many open files报错](https://zhang.ge/4541.html)
* [Baidu Cloud Ad Exchange - BCC云服务器和CDS云磁盘](https://cloud.baidu.com/doc/ADX/s/gjwvycfne)
* [linux中/etc/security/limits.conf配置文件说明](https://cloud.tencent.com/developer/article/1403636)
* [Linux中soft nproc 、soft nofile和hard nproc以及hard nofile配置](https://blog.csdn.net/zxljsbk/article/details/89153690)

