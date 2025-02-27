---
layout: post
title:  "Docker 安装 MySQL"
date:   2020-07-02 17:10:00

categories: tool
tags: docker
author: "Victor"
---

## 基础安装

1. 查看可用的 MySQL 版本，访问 [MySQL 镜像库地址](https://hub.docker.com/_/mysql)。也可以用 `docker search mysql` 查看可用镜像
2. 拉取官方的最新版本的镜像 `docker pull mysql:8.0.17`
3. 查看本地镜像是否已安装了 MySQL `docker images`
4. 运行容器 `docker run --name mysql-dev -e MYSQL_ROOT_PASSWORD=my-secret-pw -d -p 33306:3306 mysql:8.0.17`
    * -p 33306:3306：映射容器服务的 3306 端口到宿主机的 33306 端口。外部可以直接通过宿主机 ip:33306 访问到 MySQL 的服务。
    * --name mysql-dev：给容器命名为 mysql-dev
    * -d: 后台启动

5. 以通过 `docker ps` 命令查看容器的运行信息
6. 通过 mysql-cli 连接测试使用 mysql 服务 `mysql -uroot -pmy-secret-pw -h127.0.0.1 -P33306 -e 'show global variables like "max_connections"';`
7. 当启动容器端口报错时，可以通过 `netstat -lntp | grep 33306` 查看哪个程序在占用

## MySQL 容器的配置

通过 `docker exec -it <container-id> /bin/bash` 进入容器内容，会发现 MySQL 默认的配置在 `/etc/mysql/my.cnf` 目录中。

如果需要自定义配置，可以在宿主机中新建配置文件，然后挂在在容器中。

1. 在宿主机中创建目录 `mkdir -p /root/docker/[container_name]/conf.d`
2. 在该文件夹创建一个 MySQL 配置文件 `nano /root/docker/[container_name]/conf.d/my-custom.cnf`
3. 可以在这个文件中修改你的配置项，并保存。
4. 停止并删除刚才的容器，重新启动一个容器并将配置文件挂载在其中。

```conf
[mysqld]
max_connections=250
```

```bash
docker run --d --name=mysql-dev \
--env="MYSQL_ROOT_PASSWORD=my-secret-pw" \
--publish 6603:3306 \
--volume=/root/docker/mysql-dev/conf.d:/etc/mysql/conf.d \
mysql:8.0.17
```

现在再次看 MySQL 的连接数 `mysql -uroot -pmy-secret-pw -h127.0.0.1 -P33306 -e 'show global variables like "max_connections"';`

### 数据存储

默认，容器会把数据存储在容器的内部卷中。要检查卷的位置，可以使用这个命令 `docker inspect [container_name]`。

你可以看到 `/var/lib/mysql` 挂载在容器的内部卷中。

我们当然更改数据目录的位置，在宿主机上创建一个目录。然后在容器上挂载这个卷，方便其它程序的访问，也方便数据持久化。

1. `mkdir -p /storage/docker/mysql-data`
2. 然后重启容器，并将数据卷挂载上去。

```bash
docker run \
--d \
--name=[container_name] \
--env="MYSQL_ROOT_PASSWORD=my_password" \
--publish 6603:3306 \
--volume=/root/docker/[container_name]/conf.d:/etc/mysql/conf.d \
--volume=/storage/docker/mysql-data:/var/lib/mysql \
mysql:8.0.17
```

## 重启、删除容器 和 自动启动

参考 Docker 安装 Redis。

### 在其他的容器中访问 MySQL

可以创建我们的应用程序容器，使用 `--link` 参数来创建一个连接 mysql 容器，我们使用别名 mysql-db, 这将会在 mysql 容器和应用实例容器中创建一个安全的通信隧道。

```bash
docker run --link mysql-dev:mysql-db -it ubuntu:18.04 /bin/bash
```

## 相关阅读

* [Docker 安装 MySQL](https://hub.docker.com/_/mysql)
* [MySQL Docker Container Tutorial: How To Set Up & Configure](https://phoenixnap.com/kb/mysql-docker-container)
