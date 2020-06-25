---
layout: post
title:  "Multi-Stage Docker Builds + Setting the timezone Alpine"
date:   2020-06-18 17:10:00

categories: go
tags: debug
author: "Victor"
---

为了给镜像减少体积，打算分布构建镜像，本来没什么问题，但是项目中用到了时区的功能，如果镜像中没有设置时区就会让程序在部署之后执行的结果和预期不符。

网上一般给的方案是：

```dockerfile
FROM alpine:3.6
RUN apk add --no-cache tzdata
ENV TZ Asia/Shanghai
```

因为 alpine 默认没有时区，所以建议的是设置 Docker 的时区。当然方法也不限于上面一种，还可以看看 [校正alpine镜像的时区](https://blog.csdn.net/benben_2015/article/details/103091102)

As the wiki said: https://wiki.alpinelinux.org/wiki/Setting_the_timezone, You can now remove the other timezones using "apk del tzdata".

```dockerfile
FROM alpine:3.6
RUN apk add tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone \
    && apk del tzdata
```

同事的方案：

```dockerfile
FROM golang:1.14.3-alpine3.11 AS build

WORKDIR /go/release
ADD . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -mod=vendor -ldflags "-w -s" -o gateway cmd/server/main.go

FROM alpine:3.12.0
ENV GOROOT=/go
COPY --from=build /usr/local/go/lib/time/zoneinfo.zip /go/lib/time/zoneinfo.zip
COPY --from=build /go/release/gateway /
COPY --from=build /go/release/web /web
COPY --from=build /go/release/pkg/initializer/ipip/ipipfree.ipdb /

ENTRYPOINT ["/gateway"]
```

## 相关阅读

### 时区

* [Linux查看设置系统时区](https://www.cnblogs.com/kerrycode/p/4217995.html)
* [Multi-Stage Docker Builds for Creating Tiny Go Images](https://medium.com/travis-on-docker/multi-stage-docker-builds-for-creating-tiny-go-images-e0e1867efe5a)
* [Setting the timezone](https://wiki.alpinelinux.org/wiki/Setting_the_timezone)

### Docker

* [Dockerfile用法详解](https://www.jianshu.com/p/ae476d193b29)
* [Dockerfile文件详解](https://www.cnblogs.com/panwenbin-logs/p/8007348.html)
* [Speeding up Go Modules for Docker and CI](https://evilmartians.com/chronicles/speeding-up-go-modules-for-docker-and-ci)
* [](https://www.callicoder.com/docker-golang-image-container-example/)
