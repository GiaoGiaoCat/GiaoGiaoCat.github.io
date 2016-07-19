---
layout: post
title:  "brew install mysql on mac os 10.10"
date:   2014-10-27 11:40:00
categories: tool
tags: mac database
author: "Victor"
---

### Step-by-step

```bash
brew remove mysql
brew cleanup
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
rm ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
sudo rm -rf /usr/local/var/mysql #删除全部数据库
brew install mysql
# Inject launch agent
unset TMPDIR
# Set up databases to run as your user account
mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
```

然后进行安全设置，注意下面命令中的 mysql 版本要和目前安装的版本一致

```bash
/usr/local/Cellar/mysql/5.5.10/bin/mysql_secure_installation
```

用下面的命令自动在系统启动的时候加载 mysql

```bash
# To have launchd start mysql at login:
ln -sfv /usr/local/opt/mysql/*plist ~/Library/LaunchAgents

# start
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
# stop
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```

下面两个命令可以给你提供很大的帮助

```bash
brew info mysql
mysql --help
```

### 配置 my.cnf

如果找不到 /etc/my.cnf 也不用怕，自己创建一个，然后复制如下内容即可

```
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8

[mysqld]
collation-server = utf8_unicode_ci
character-set-server = utf8
init-connect ='SET NAMES utf8'
max_allowed_packet = 64M
bind-address = 127.0.0.1
port = 3306
socket = /tmp/mysql.sock
```

### How to Disable Strict SQL Mode in MySQL 5.7

先用 `mysql --help` 查看，发现 MySQL 从如下位置读取配置文件 `/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf`

`sudo cp $(brew --prefix mysql)/support-files/my-default.cnf ~/.my.cnf`

编辑该文件添加如下

```
[mysqld]
sql_mode=IGNORE_SPACE,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

重启 MySQL

### 相关链接

* [brew install mysql on mac os](http://stackoverflow.com/questions/4359131/brew-install-mysql-on-mac-os)
* [mac os 安装 mysql总是出现cannot find mysql.sock](https://ruby-china.org/topics/794)
* [在 Mac 下用 Homebrew 安装 MySQL](http://blog.neten.de/posts/2014/01/27/install-mysql-using-homebrew/)
* [Disable Services in OSX (services.msc)](http://apple.stackexchange.com/questions/105892/disable-services-in-osx-services-msc)
* [How to Disable Strict SQL Mode in MySQL 5.7](https://serverpilot.io/community/articles/how-to-disable-strict-mode-in-mysql-5-7.html)
