---
layout: post
title:  "使用 rails-dev-box 搭建 Vagrant 环境"
date:   2016-01-16 22:30:00
categories: tool
tags: virtual_machine
author: "Victor"
---

当学会 Vagrant 之后，下一步想要的就是避免每次启动新项目都要自己重头搭建一套环境。

[rails-dev-box](https://github.com/rails/rails-dev-box) 就是帮我们解决这个痛点的项目。该项目的唯一目的就是帮我们自动搭建一套 Rails 的开发环境。

### What's In The Box

* Development tools
* Git
* Ruby 2.3
* Bundler
* SQLite3, MySQL, and Postgres
* Databases and users needed to run the Active Record test suite
* System dependencies for nokogiri, sqlite3, mysql, mysql2, and pg
* Memcached
* Redis
* RabbitMQ
* An ExecJS runtime

### How To Build The Virtual Machine

```bash
cd ~/Documents/Vagrant/
git clone https://github.com/rails/rails-dev-box.git
cp Vagrantfile bootstrap.sh ../fiverails # 把 rails-dev-box 的核心文件复制到项目目录
vagrant up
vagrant ssh
cd /vagrant
bundle
```

上面的步骤根据网络情况，可能需要修改一下 `bootstrap.sh` 修改 GEM 源：

```
echo installing Bundler
echo 'gem: --no-ri --no-rdoc' >~/.gemrc
gem source -r https://rubygems.org/
gem source -a https://ruby.taobao.org/
gem install bundler -N >/dev/null 2>&1
```

然后就搞定了

```
bin/rails server -b 0.0.0.0
```

### Faster Rails test suites

默认的文件夹共享机制很方便，并且可以很稳定的工作在各种 Vagrant 版本下。但是仍然有一些可替代的方案让性能更好。

### rsync

Vagrant 1.5 实现了 [sharing mechanism based on rsync](https://www.vagrantup.com/blog/feature-preview-vagrant-1-5-rsync.html)。它只通过发送文件差异，把机器之间传输的数据总量降低到最低限度。

要启用这一特性只需要在 **Vagrantfile** 中添加

```bash
config.vm.synced_folder '.', '/vagrant', type: 'rsync'
```

默认情况下，文件夹仅在 `vagrant up`，`vagrant rsync`，`vagrant reload` 的时候才同步。

这是 Rsync 方式和默认的同步方式以及 NFS 的区别，如果你需要持续同步变化，需要执行 `vagrant rsync-auto`。

### NFS

对于 Mac OS X 和 Linux 的用户更推荐使用 NFS 同步方式来提高文件同步速度。

NFS 服务已经默认在 Mac OS X 中安装。

```
config.vm.synced_folder '.', '/vagrant', type: 'nfs'
config.vm.network 'private_network', ip: '192.168.50.4' # ensure this is available
```

然后在 `vagrant up` 之后就会自动生效。

## 参考

* [rails-dev-box](https://github.com/rails/rails-dev-box)
* [railsbox](https://railsbox.io/)
