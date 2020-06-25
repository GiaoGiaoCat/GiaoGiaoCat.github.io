---
layout: post
title:  "docker compose 配置 golang + redis"
date:   2020-06-18 17:10:00

categories: tool
tags: docker
author: "Victor"
---

## 基础

docker-compose 不仅可以根据配置文件 docker-compose.yml 自动创建网络，启动响应的容器实例，也可以根据配置文件删除停止和删除容器实例，并删除对应的网络。

docker-compose将所管理的容器分为3层结构：`project service container`。

docker-compose.yml 组成一个 project，project 里包括多个 service，每个 service 定义了容器运行的镜像（或构建镜像），网络端口，文件挂载，参数，依赖等，每个 service 可包括同一个镜像的多个容器实例。

### 基础命令

* `docker-compose up` 前台运行
* `docker-compose up -d` 后台运行
* `docker-compose start` 启动
* `docker-compose stop` 停止
* `docker-compose down` 停止并移除容器
* `docker-compose ps` 列出项目中目前的所有容器
* `docker-compose logs` 查看服务容器的输出
* [更多命令](https://docs.docker.com/compose/reference/overview/)

### 实际使用

首先看一下 `docker version` 因为它决定了 docker-compose.yml 中 version 的值，具体可以看 [Compose file versions and upgrading](https://docs.docker.com/compose/compose-file/compose-versioning/)。

```yaml
version: '3'                      # 定义版本，根据本机 Docker 的版本选择
services:
  redis-db:                       # 名称，它也是 network 中 DNS 名称
    image: redis:latest           # 镜像，如果像自定义镜像可以不指定这个参数，而用 build
    container_name: redis-gateway
    restart: always
    ports:
      - "16379:6379"

  gateway:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - redis-db
    environment:
      SSP_REDIS_HOST: redis-db
      SSP_REDIS_PORT: 6379
      SSP_SERVER_PORT: 8080
      SSP_SERVER_MODE: release
      SSP_IPIP_DB: "./ipipfree.ipdb"
    volumes:
      - "$PWD/scripts/wait-for-it.sh:/wait-for-it.sh:ro"
    entrypoint: ["./wait-for-it.sh", "redis-db:6379", "--", "/gateway"]
```

### 启动顺序

[官方文档](https://docs.docker.com/compose/startup-order/) 有讲容器不会在 Compose 启动时根据 depends_on 去决定等待所有容器依次准备好。具体的原因官方也有陈述。

等待数据库准备就绪的问题实际上只是分布式系统需要解决的问题之一。 在生产环境中，数据库可能随时不可用。你的代码需要能够应对这些类型的故障，可以自动重连数据库之类的。

但是如果我们不需要考虑这种情况，那么也有一些工具来帮我们保证启动顺序：

* [wait-for-it](https://github.com/vishnubob/wait-for-it/issues/57)
* [dockerize](https://github.com/jwilder/dockerize)
* [wait-for](https://github.com/Eficode/wait-for)

使用方法也很简单：

1. 在代码库中添加 wait-for-it.sh 执行 `chmod 755 wait-for-it.sh`
2. 修改 docker-compose.yml 在 app service 中添加对 require services 的监测代码
3. 如果镜像是基于 alpine 构建的，还需要修改 Docker 添加 Bash 支持

```yaml
redis-db:
  image: redis:latest
  ports:
    - "16379:6379"
gateway:
  depends_on:
    - redis-db
  volumes:
    - "./wait-for-it.sh:/wait-for-it.sh:ro"
  entrypoint: ["./wait-for-it.sh", "redis-db:6379", "--", "/gateway"]
```

```yaml
...
FROM alpine:3.12.0

RUN apk add --no-cache bash
...
```

这里需要注意的是 redis-db 中设置的端口是 `16379:6379`，其中 16379 是导出到宿主机的端口，也就是说宿主机可以通过该端口连接 Redis 进行管理。而 compose 内部的 service 之间还是使用 6379 进行通信。这一点和单独启动两个容器进行通信的时候不一样。

## 相关阅读

* [Docker Compose](https://www.runoob.com/docker/docker-compose.html)
* [docker-compose教程（安装，使用, 快速入门）](https://blog.csdn.net/pushiqiang/article/details/78682323)
* [附录（redis.conf）](https://www.cnblogs.com/xpengp/p/12713374.html)
* [docker compose 常用命令](https://www.cnblogs.com/yyxianren/p/10894708.html)
* [Our Basic Docker Compose File](https://serversforhackers.com/dockerized-app/docker-compose)
* [Docker環境下でGoとRedisでAPIを実装する (Part.1)](https://qiita.com/Morero/items/473bc26ce2200c6a6fc6)
* [使用go mod结合docker分层缓存进行自动CI/CD](https://juejin.im/post/5c887c105188257edb45e5b1)
