---
layout: post
title:  "Docker 安装 InfluxDB"
date:   2020-10-17 17:10:00

categories: tool
tags: docker
author: "Victor"
---

## 基础安装

1. 拉取官方的最新版本的镜像 `docker pull influxdb`
2. 查看本地镜像是否已安装了 influxdb `docker images ls`
3. 运行容器 `docker run -itd --name influxdb-dev -p 8086:8086 -v $PWD:/var/lib/influxdb influxdb`
    * -p 8086:8086：映射容器服务的 8086 端口到宿主机的 8086 端口。8086 是 InfluxDB 的 HTTP API 端口
    * --name influxdb-dev：给容器命名为 influxdb-dev
    * -it：交互式
    * -d: 后台启动
    * -v $PWD:/var/lib/influxdb 把容器内的 /var/lib/influxdb 挂载到当前目录
    * -e userame="jaxlove" 设置环境变量
    * --rm 退出容器以后，这个容器就被删除了，方便在临时测试使用，不能与 -d 同时使用
4. 以通过 `docker ps` 命令查看容器的运行信息
5. 也可以自定义一个被挂载的目录

```bash
docker run -d -p 8086:8086 -v influxdb:/var/lib/influxdb influxdb
```

注意，有些文章教你还要导出 8083 端口，因为新版本移除了 Web 控制台，所以我们省略这个端口就行。

因为我并不想挂载数据，所以我本地执行的是 `docker run -d -p 8086:8086 --name influxdb-dev influxdb`

## 配置

InfluxDB 同样可以使用配置和环境变量来控制它的参数，

1. 生成默认的配置文件：`docker run --rm influxdb influxd config > influxdb.conf`
2. 现在把刚才生成的配置文件加载进来，也可以把 $PWD 改成你存放 influxdb.conf 文件的目录。

```bash
docker run -p 8086:8086 \
      -v $PWD/influxdb.conf:/etc/influxdb/influxdb.conf:ro \
      influxdb -config /etc/influxdb/influxdb.conf
```

## 使用

```bash
docker exec -it influxdb-dev bash
```

进入 `/usr/bin` 目录，这里面有 Influxdb 的工具：

```bash
find | grep influx
./influx_tsm
./influx_stress
./influxd
./influx_inspect
./influx
```

查看 InfluxDB 版本，`./influx -version`。
进入 InfluxDB 客户端命令行：

```bash
./influx
> show databases
> create database my_test // 创建数据库
> drop database [db_name] //  删除数据库
> use my_test // 使用数据库
> show measurements // 显示表列表
> select * from test_measurement // 查看表
> drop measurement [test_measurement] // 删除表
```

## 相关阅读

* [Quick reference](https://hub.docker.com/_/influxdb)
* [influxdb安装和学习](https://www.cnblogs.com/woshimrf/p/docker-influxdb.html)
* [Docker下安装Influxdb-1.6.1和Grafana5.2.2](https://www.cnblogs.com/LUA123/p/9507029.html)
