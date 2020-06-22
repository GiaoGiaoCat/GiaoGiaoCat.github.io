---
layout: post
title:  "Docker 安装 Redis"
date:   2020-06-18 17:10:00

categories: tool
tags: docker
author: "Victor"
---

## 基础安装

1. 查看可用的 Redis 版本，访问 [Redis 镜像库地址](https://hub.docker.com/_/redis?tab=tags)。也可以用 `docker search redis` 查看可用镜像
2. 拉取官方的最新版本的镜像 `docker pull redis:latest`
3. 查看本地镜像是否已安装了 redis `docker images`
4. 运行容器 `docker run -itd --name redis-dev -p 6379:6379 redis`
    * -p 6379:6379：映射容器服务的 6379 端口到宿主机的 6379 端口。外部可以直接通过宿主机 ip:6379 访问到 Redis 的服务。
    * --name redis-dev：给容器命名为 redis-dev
    * -it：交互式
    * -d: 后台启动
    * -e userame="jaxlove" 设置环境变量
    * --rm 退出容器以后，这个容器就被删除了，方便在临时测试使用，不能与 -d 同时使用

5. 以通过 `docker ps` 命令查看容器的运行信息
6. 通过 redis-cli 连接测试使用 redis 服务，下面三种都可以
    * `docker exec -it redis-dev redis-cli` 推荐这个
    * `docker exec -it redis-dev /bin/bash`
    * `docker exec -it redis-dev bash`
7. 当启动容器端口报错时，可以通过 `netstat -lntp | grep 6379` 查看哪个程序在占用

```bash
docker run -itd --name redis-dev -p 6379:6379 redis
```

### 重启容器

重启电脑后，容器需要手动重启。

1. `docker ps -a`
2. 使用命令对镜像从起 `docker restart <CONTAINER ID>`

### 自动启动

在以前的版本中，我们需要自定义一个 plist，然后通过加载它的方法来自启动 docker。自从安装了 [Docker for Mac](https://docs.docker.com/docker-for-mac/) 也就不用麻烦了。

我要做的是每次开机自动重启 Redis 的容器。

* 在运行 docker 容器时可以加 `--restart=always` 参数来保证每次 docker 服务重启后容器也自动重启。
* 如果已经启动了则可以使用如下命令 `docker update --restart=always <CONTAINER ID>`

```bash
docker run -itd --restart=on-failure:10 --name redis-dev -p 6379:6379 redis
```

Docker容器的重启策略如下：

* `no` 默认策略，在容器退出时不重启容器
* `on-failure` 在容器非正常退出时（退出状态非0），才会重启容器
* `on-failure:3` 在容器非正常退出时重启容器，最多重启3次
* `always` 在容器退出时总是重启容器
* `unless-stopped` 在容器退出时总是重启容器，但是不考虑在 Docker 守护进程启动时就已经停止了的容器

## 访问

### 默认启动

```bash
docker run --name redis-dev -d redis
```

之后就可以通过客户端程序连接 127.0.0.1:6379 来访问了。

### 局域网访问配置

> 因为 redis 默认配置你会发现只能够本地连接，不能进行远程访问，使用 Redis Desktop Manager 连接都会报错，因此需要手动挂载 redis 配置文件。

但是我不确定，因为我今天没带两个机器。而且我安装之后，宿主机默认就能访问 Redis。等遇到问题有需要的看下面的文章吧。

* [Docker 安装 Redis 完整过程及配置远程连接&踩坑注意事项](https://blog.csdn.net/u010358168/article/details/97143703)
* [Docker 安装 Redis 让宿主机可以访问](https://www.cnblogs.com/shihuibei/p/10663112.html)
* [Docker 安装 Redis （Redis 配置）](https://cloud.tencent.com/developer/article/1562815)

### 在其他的容器中访问 Redis

按照这篇文章 [在Docker中运行Reids服务](https://doc.yonyoucloud.com/doc/chinese_docker/examples/running_redis_service.html) 配置的。

可以创建我们的应用程序容器，使用 `--link` 参数来创建一个连接 redis 容器，我们使用别名 redis-db, 这将会在 redis 容器和应用实例容器中创建一个安全的通信隧道。

```bash
docker run --link redis-dev:redis-db -it ubuntu:18.04 /bin/bash
```

进入我们刚才创建的容器，我们需要安装 redis 的 `redis-cli` 的二进制包来测试连接。先查看下 we b应用程序容器的环境变量，我们可以用我们的 ip 和端口来连接 redis 容器。

```bash
env
. . .
REDIS_DB_NAME=/crazy_brahmagupta/redis-db
REDIS_DB_ENV_REDIS_VERSION=6.0.5
REDIS_DB_PORT=tcp://172.17.0.2:6379
REDIS_DB_PORT_6379_TCP_PROTO=tcp
REDIS_DB_PORT_6379_TCP_PORT=6379
REDIS_DB_PORT_6379_TCP_ADDR=172.17.0.2
. . .
```

我们可以看到有一组 REDIS_DB 为前缀的环境变量列表，REDIS_DB 来自指定别名连接我们的现在的容器，使用 REDIS_DB_PORT_6379_TCP_ADDR 变量连接到 Redis 容器。

```bash
apt-get update
apt-get -y install redis-server
service redis-server stop
# redis-cli -h redis-db
# redis-cli -h 172.17.0.2
redis-cli -h $REDIS_DB_PORT_6379_TCP_ADDR
. . .
redis 172.17.0.33:6379> set docker awesome
OK
redis 172.17.0.33:6379> get docker
"awesome"
redis 172.17.0.33:6379> exit
```

如果嫌弃上面的步骤太麻烦，可以直接用下面的命令。

```bash
docker run -it --link redis-dev --rm redis redis-cli -h redis-dev -p 6379
```

### 自定义配置和数据持久化

数据持久化存储到宿主机。

```bash
docker run --name redis-dev2 -d -v ~/data/redis:/data redis redis-server --appendonly yes
```

* `--appendonly yes` 用于打开 redis 的数据持久化存储
* `-v ~/data/redis:/data` 用于将宿主机的目录映射到容器对应的数据存储目录

自定义配置文件。

[官网](https://redis.io/)下载指定版本的 Redis 源码包，解压后获取默认配置文件 [redis.conf](http://download.redis.io/redis-stable/redis.conf)。然后在 Redis 容器启动时如下操作：

```bash
# ~/myredis/conf/redis.conf 对应宿主机配置文件位置
docker run -v ~/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf --name myredis3 redis redis-server /usr/local/etc/redis/redis.conf
```

你可以需要修改如下参数

* bind
* protected-mode
* requirepass
* daemonize 这里需要特别注意，设置成 No，下文有讲。

## 使用 docker compose 编排

上面是基本使用方式，为便于管理，我们一般使用 docker compose 来统一创建、管理各服务。

新建一个目录 redis 存放，结构如下：

```bash
➜  tree -L 2 redis
.
├── conf
│   └── redis.conf # 默认配置文件
├── data
│   └── dump.rdb  # 数据持久化文件
└── docker-compose.yml
```

docker-compose.yml 文件如下：

```yaml
# Redis
# https://redis.io
# https://hub.docker.com/_/redis
version: '2'
services:
  redis:
    image: redis:latest
    container_name: redis-dev
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data:rw
      - ./conf/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    # privileged: true
```

> 注意：启动前，注意修改配置文件中的连接密码（requirepass）和 IP 绑定（bind）两个属性，尤其是当使用 redis-cli 或其它客户端工具连接发生异常时。

启动并测试连接：

```bash
docker compose up
docker exec -it ${Redis 容器 ID} redis-cli -h 127.0.0.1 -p 6379
```

> 在较高版本的 redis.conf 配置文件中，属性 protected-mode yes 被默认开启，以保证 Redis 的安全性。
> 所以必须给 Redis 设置密码才可以连接。
> 如果只是测试用而不需要密码，则修改此属性的值为 no ，并且属性 requirepass xxx 不要打开即可。

## 相关阅读

* [Docker 安装 Redis](https://www.runoob.com/docker/docker-install-redis.html) 基础安装
* [Docker容器的重启策略及docker run的--restart选项详解](https://blog.csdn.net/taiyangdao/article/details/73076019)
* [基于 Docker 的 Redis 服务](https://blog.sqiang.net/post/3425556216.html)
* [官方 Docker run reference](https://docs.docker.com/engine/reference/run/)
* [Docker Compose](https://www.runoob.com/docker/docker-compose.html)
* [docker-compose教程（安装，使用, 快速入门）](https://blog.csdn.net/pushiqiang/article/details/78682323)
