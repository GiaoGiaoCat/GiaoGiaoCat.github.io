---
layout: post
title:  "Mac OS 升级 10.9 后 finder 变慢的解决方法"
date:   2014-08-29 14:40:00
categories: tool
tags: mac
author: "Victor"
---

### 方法1

如果你在某个应用中，需要选取本地文件的话（邮箱里面选择附件上传），打开文件选取窗口，第一次一定会出现菊花长时间转 的情况，表现为图中圆圈选中的区域一个菊花拼命转，半分钟之后正常了，你可以选择文件了。

现在暂时性解决办法找到了：https://discussions.apple.com/thread/5495797?start=30&tstart=0


在terminal中输入如下语句：

```
sudo vi /etc/auto_master
```

然后注释掉/net开头的那行（在行首加#符）保存退出。

然后执行如下语句重载配置：

```
sudo automount -vc
```

即可解决这个问题。

不过如果你还是使用了网络存储的话，这个办法解决不了。

### 方法2

经本人探索，以下方法可以解决升级 10.9 后 finder 变慢的问题：

1. finder 偏好设置中，边栏选项卡设置中勾选上“我的全部文件”，
2. 通用选项卡设置中：开启新的 finder 窗口时选择打开“我的全部文件”。
3. 重新启动电脑。再启动 finder 看看，原先的卡涩现象是不是消除掉了。

经测试，经上述三部处理后，我的Macbook、iMac、Mac Mini全部告别了finder查看文件的卡涩现象。
