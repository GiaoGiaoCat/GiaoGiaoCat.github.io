---
layout: post
title:  "Easy Rails Development Environment With Vagrant"
date:   2016-01-15 22:30:00
categories: tool
tags: virtual_machine
author: "Victor"
---

## Getting Started

当我们进行 Rails 开发的时候，总是需要解决下面的依赖：

* 数据库服务器(MySQL，PostgreSQL)
* key-value 型存储系统(Redis，memcached)
* 后台进程(sidekiq)
* 搜索引擎(ElasticSearch)

在开发机器商安装和运行这些东西，可能很快就让电脑变得乱七八糟。当你在这台机器上开发另外一套程序的时候，可能依赖环境之间又会互相破坏。使用 Vagrant 可以在虚拟机中隔离这些环境。一旦你的电脑出现问题，在新的电脑上重建开发环境也非常简单。

Vagrant 提供了统一的安装程序配置环境：

* 使用统一的配置文件 **vagrantfile**实现对服务器的统一配置。
* 使用共享文件夹 **shared folder**实现代码编辑向“服务器”的快速提交
* 使用软件配置脚本 **Provisioning scripts** 实现服务器上的运行环境的快速建立
* 拥有标准化的虚拟机分享网络，极大缓解了分享开发环境配置时的网络带宽压力
* 可以具备一个供安装维护测试使用的可抛弃的服务器端环境。

### Installing VirtualBox and Vagrant

* VirtualBox 是一款开源虚拟机软件，可以让一台机器同时运行多个系统。
* Vagrant 是一个基于 Ruby 的工具，用于创建和部署虚拟化开发环境。它使用 VirtualBox 虚拟化系统，使用 Chef 创建自动化虚拟环境。
* vagrant-manager 可以很方便的在菜单栏管理所有虚拟机。

```bash
brew tap caskroom/cask
brew install brew-cask
brew cask search virtualbox
brew cask install vagrant
brew cask install vagrant-manager
```

## Setting Up Your Rails Stack

### The Vagrantfile

Vagrant 初始化成功后，会在初始化的目录里生成一个 Vagrantfile 的配置文件，可以修改配置文件进行个性化的定制。

```bash
cd ~/Documents/Dev/ # 切换目录
vagrant init  # 初始化，默认环境名称 base，也可以在 init 后面添加自定义名称，比如：vagrant init haha
atom Vagrantfile # 编辑配置文件
```

每个 Vagrant 开发环境都需要一个系统镜像。可以在 https://atlas.hashicorp.com/search 选择一个合适的镜像，这里我们用 **bento/ubuntu-14.04**

```ruby
config.vm.box = "bento/ubuntu-14.04"
```

Vagrant 默认是使用端口映射方式将虚拟机的端口映射本地从而实现类似 http://localhost:80 这种访问方式，因为我们是开发 Rails 所以直接把 3000 端口转发过去就好。

```ruby
config.vm.network "forwarded_port", guest: 3000, host: 3000
```

不过这个端口转发方式其实很麻烦，host-only 模式显得方便多了。将下面这行的注释去掉（移除 #）并保存：

```ruby
config.vm.network :private_network, ip: "192.168.33.10"
```

重启虚拟机，这样我们就能用 **192.168.33.10** 访问这台机器了，你可以把 IP 改成其他地址，只要不产生冲突就行。

```bash
vagrant up # 启动虚拟机，第一次更新镜像，根据网络状况可能需要较长时间
```

你会看到终端显示了启动过程，启动完成后，我们就可以用 SSH 登录虚拟机了，剩下的步骤就是在虚拟机里配置你要运行的各种环境和参数了。

```
vagrant ssh
```

### Installing Ruby Using RVM

跟在本地执行安装并没啥不同：

```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
sudo apt-get install curl
\curl -sSL https://get.rvm.io | bash -s head
source /home/vagrant/.rvm/scripts/rvm
rvm list known
rvm install 2.3.0
ruby -v # 确认 ruby 版本正确
gem sources -r https://rubygems.org/ # 删除官方镜像，GFW 呵呵
gem sources -a https://ruby.taobao.org # 改用淘宝镜像
echo "gem: --no-ri --no-rdoc" > ~/.gemrc
gem install bundler --no-ri --no-rdoc
```

### Installing MySQL, PostgreSQL and Redis

```
sudo apt-get install mysql-server mysql-client libmysqlclient-dev
```

```bash
sudo apt-get install postgresql-contrib postgresql-9.3 libpq-dev postgresql-client postgresql-server-dev-9.3
sudo su postgres -c "createuser vagrant -s"
psql # 验证一下
$ psql: FATAL:  database "vagrant" does not exist
```

```bash
mkdir tmp
cd tmp
curl http://download.redis.io/redis-stable.tar.gz -o redis-stable.tar.gz
tar xfz redis-stable.tar.gz
cd redis-stable/
make
sudo make install
```

```bash
sudo mkdir /etc/redis /var/redis
sudo cp utils/redis_init_script /etc/init.d/redis_6379
sudo cp redis.conf /etc/redis/6379.conf
sudo vi /etc/redis/6379.conf
```

修改 redis 配置文件

```bash
daemonize yes # 打开守护进程
pidfile /var/run/redis_6379.pid # 修改一下进程 pid
bind 127.0.0.1 # 取消注释，绑定本地 IP
logfile "/var/log/redis_6379.log"
dir /var/redis/6379
```

保存后创建数据文件夹

```bash
sudo mkdir /var/redis/6379
sudo update-rc.d redis_6379 defaults
sudo service redis_6379 start
redis-cli
```

各种类库

```
sudo apt-get install build-essential git-core curl libssl-dev libreadline5 libreadline-gplv2-dev zlib1g zlib1g-dev libmysqlclient-dev libcurl4-openssl-dev libxslt-dev libxml2-dev libffi-dev libgmp3-dev
```

Rails 5 的安装请参考文章最后的附录。

```bash
sudo apt-get install git
git clone --depth 1 git@github.com:rails/rails.git
cd rails
bundle
railties/exe/rails new --edge fiverails
cd ..
mv fiverails /vagrant # 把项目文件放在同步文件夹中
cd /vagrant/fiverails
bundle install
```

可能遇到的问题 `rails s` 无法启动，查看 `rails -v`：

```
/home/vagrant/.rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/rubygems/dependency.rb:319:in `to_specs': Could not find 'railties' (>= 0.a) among 119 total gem(s) (Gem::LoadError)
Checked in 'GEM_PATH=/home/vagrant/.rvm/gems/ruby-2.3.0:/home/vagrant/.rvm/gems/ruby-2.3.0@global', execute `gem env` for more information
  from /home/vagrant/.rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/rubygems/dependency.rb:328:in `to_spec'
  from /home/vagrant/.rvm/rubies/ruby-2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_gem.rb:65:in `gem'
  from /home/vagrant/.rvm/gems/ruby-2.3.0/bin/rails:22:in `<main>'
  from /home/vagrant/.rvm/gems/ruby-2.3.0/bin/ruby_executable_hooks:15:in `eval'
  from /home/vagrant/.rvm/gems/ruby-2.3.0/bin/ruby_executable_hooks:15:in `<main>'
```

这时候需要再次尝试安装 `gem install railties` 就可以。

```bash
rails s -b 0.0.0.0
```

### Autostart Rails and Sidekiq

每次开发时候，人工启动 rails 服务和各种后台进程非常麻烦而且很容易出错。下面介绍如何自动来完成这项工作：

首先修改 Gemfile `gem 'foreman'` 然后 `bundle` 并在项目根目录下创建 **Procfile**

```bash
# Procfile
web: bundle exec rails s -b 0.0.0.0 -p 3000
# worker: bundle exec sidekiq
```

进行验证 `foreman check`。接下来按照正常服务器的部署方法进行设置，只复制命令，具体可以看参考链接。

```bash
rvm wrapper 2.3.0
ll ~/.rvm/wrappers/ruby-2.3.0
echo RVM_WRAPPERS=$rvm_path/wrappers/$(rvm current) > .env
$lrwxrwxrwx 1 vagrant vagrant 43 Jan 17 04:33 /home/vagrant/.rvm/wrappers/ruby-2.3.0 -> /home/vagrant/.rvm/gems/ruby-2.3.0/wrappers/ # 这里是回显
cat .env
$ RVM_WRAPPERS=/home/vagrant/.rvm/wrappers/ruby-2.3.0 # 这里是回显
```

修改 **Procfile**

```bash
# Procfile
web: $RVM_WRAPPERS/bundle exec rails s -b 0.0.0.0 -p 3000
# worker: $RVM_WRAPPERS/bundle exec sidekiq
```

添加服务进程并启动

```bash
rvmsudo foreman export upstart /etc/init -a rails -u vagrant
sudo service rails start
sudo service rails restart
sudo service rails stop
ps aux | grep rails
```

## 常用命令

```ruby
$ vagrant init  # 初始化
$ vagrant up  # 启动虚拟机
$ vagrant halt  # 关闭虚拟机
$ vagrant reload  # 重启虚拟机
$ vagrant ssh  # SSH 至虚拟机
$ vagrant status  # 查看虚拟机运行状态
$ vagrant destroy  # 销毁当前虚拟机
```

## 推荐插件

* [Atom Vagrant](https://atom.io/packages/vagrant) Vagrant commands for Atom editor.
* [atom-vagrant-manager](https://atom.io/packages/atom-vagrant-manager) A manager for start and stopping vagrant boxes.

## 参考

* [VAGRANT 中文文档](http://tangbaoping.github.io/vagrant_doc_zh/v2/) 建议阅读一遍文档
* [为什么要使用Vagrant](http://www.ituring.com.cn/article/131600)
* [使用 Vagrant 打造跨平台开发环境](http://segmentfault.com/a/1190000000264347)
* [Mac OS X Setup Guide](http://sourabhbajaj.com/mac-setup/Vagrant/README.html)
* [Bento](http://chef.github.io/bento/)。
* [Setup Ruby On Rails on Ubuntu 14.10 Utopic Unicorn](https://gorails.com/setup/ubuntu/14.10)
* [安装 Rails 5](/rails/learn-rails-5/)
* [Ubuntu 14.04 LTS 服务器配置](/tool/rails101/)
* [Using RVM and Ruby-based services that start via init.d or upstart](https://rvm.io/deployment/init-d)
