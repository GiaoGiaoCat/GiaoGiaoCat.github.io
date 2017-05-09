---
layout: post
title:  "CentOS 7 服务器配置"
date:   2016-06-20 12:00:00
categories: tool
tags: server
author: "Victor"
---

关于如何配置 ssh 和 sshd 的问题，请找 Blog 中的相关文章，这里不再敖述。

## root 执行

### 语言问题

```bash
vi /etc/environment

LANG=en_US.utf-8
LC_ALL=en_US.utf-8
```

### 安装依赖

```bash
yum -y update
yum grouplist
yum groupinstall -y 'Development Tools'

# Enable EPEL Repository
sudo su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'
# Update everything, once more.
yum -y update

yum install -y curl-devel nano sqlite-devel libyaml-devel
yum install -y openssl-devel readline-devel zlib-devel

yum install -y nodejs
yum install -y npm
npm install bower -g
```

### 安装 Nginx

```bash
yum install epel-release
yum install nginx
systemctl start nginx
systemctl enable nginx
```

If you are running a firewall, run the following commands to allow HTTP and HTTPS traffic:

```bash
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

### 添加 deployer 用户

```bash
adduser deployer
passwd deployer
gpasswd -a deployer wheel
```

给用户配置 ssh 登录密钥，以下命令在本地开发机器上执行：

```
ssh-copy-id deployer@SERVER_IP_ADDRESS
```

### 安装 MySQL

Staging 机器，可以把 数据库 放在 ECS 而不用单开一个 RDS

```bash
yum install mariadb-server mariadb
systemctl start mariadb.service
systemctl enable mariadb.service
yum install mysql-devel
```

### 安装 memcached

Staging 机器，可以把 Memcached 放在 ECS

```bash
yum install memcached
systemctl status memcached
systemctl enable memcached # Be sure that Memcached starts at boot:
```

## deployer 执行

### 安装 Ruby

```bash
git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
git clone git://github.com/jamis/rbenv-gemset.git  ~/.rbenv/plugins/rbenv-gemset
git clone git://github.com/sstephenson/rbenv-gem-rehash.git ~/.rbenv/plugins/rbenv-gem-rehash
git clone git://github.com/rkh/rbenv-update.git ~/.rbenv/plugins/rbenv-update
git clone git://github.com/AndorChen/rbenv-china-mirror.git ~/.rbenv/plugins/rbenv-china-mirror
```

```bash
vi ~/.bash_profile
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"
```

```bash
rbenv install --list
rbenv install 2.3.0
rbenv global 2.3.0
```

退出再重新登录

### 配置 Gem

```bash
echo "gem: --no-ri --no-rdoc" > ~/.gemrc
gem install bundler
```

### MySQL

```
gem install mysql2 -v '0.4.4'
```

### Install nokogiri gem on CentOS with prerequesites

在使用 mina 部署的时候，如果 nokogiri 的 gem 无法安装，可以先用 deployer 账户登录之后，安装如下依赖

```bash
sudo yum install libxslt-devel libxml2-devel
```

### 其他

```
ssh-keygen
ssh-keyscan -H github.com >> ~/.ssh/known_hosts
cat ~/.ssh/id_rsa.pub # 复制到 github 的部署机器人身上
service sshd restart
```

### Nginx 无法访问 assets 文件夹下面的文件

1. 把 assets 在 mina 中配置成 shared 文件
2. `chmod o+x /home/deployer` [参考此文](https://stackoverflow.com/questions/6795350/nginx-403-forbidden-for-all-files)

## 相关阅读

* [Initial Server Setup with CentOS 7](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7)
* [How To Deploy With Mina: Getting Started](https://www.digitalocean.com/community/tutorials/how-to-deploy-with-mina-getting-started)
* [rbenv 使用指南](https://ruby-china.org/wiki/rbenv-guide)
* [Install MySQL Server on CentOS](https://support.rackspace.com/how-to/installing-mysql-server-on-centos/)
* [How to Install Memcached on CentOS 7](https://www.liquidweb.com/kb/how-to-install-memcached-on-centos-7/)
* [How To Install Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)
