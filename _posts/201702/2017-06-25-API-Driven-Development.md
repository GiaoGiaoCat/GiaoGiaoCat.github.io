---
layout: post
title:  "API Driven Development with APIary.io"
date:   2017-06-25 11:40:00
categories: tool
tags: testing
author: "Victor"
---

## 前言

我们常见的开发方式主要是 TDD(Threat-Driven AppDevelopment) 和 BDD(Boss-Driven AppDevelopment)。

TDD 的经典场景：

```
产品：这个功能周一上面要验收，要不周末加加班吧。
程序员：好吧（CNM）
```

BDD 的经典场景：

```
老板：你下周给我做一个类似微信的这个功能。
程序员：好的老板（CNM）
```

## API Driven Development

### 从 API 文档开始

[apiary.io](https://apiary.io/) 是使用 BluePrint 格式的一款文档工具，它支持实时预览功能。

假设我们的 `Blog` API 项目只有 2 个服务接口，`GET /article` 和 `POST /article`。

```blueprint
FORMAT: 1A

# blog

Blog is a simple API allowing consumers...

## Article Collection [/article]

### List All Articles [GET]

+ Response 200 (application/json)

        [
            {
                "title": "Article 1",
                "content": "content"
            },
            {
                "title": "Article 2",
                "content": "content"
            }
        ]

### Create a New Article [POST]

You may create your own article using this action. It takes a JSON
object containing...

+ Request (application/json)

        {
            "title": "My new post",
            "content": "new article content",
        }

+ Response 201 (application/json)

    + Body

            {
               "status":"created"
            }
```

### 创建 API 应用

我们的目标是期望程序员实现的 API 和上面描述的文档一致。为了实现这个目标，需要引入 [Dredd](https://github.com/apiaryio/dredd)。

Dredd 是一个 HTTP API 测试框架。它会阅读你的 API 文档，然后验证你的 API 实现的响应是否和文档描述的一致。

假设我们的文档地址是 `https://jsapi.apiary.io/apis/blog41.apib` 开发地址是 `http://blog.dev`

在项目的根目录执行如下命令

```
dredd https://jsapi.apiary.io/apis/blog41.apib http://blog.dev
```

因为我们还没有实现任何 API 所以，返回的信息会是和文档描述的返回值一样，并提示 `complete: 0 passing, 2 failing, 0 errors, 0 skipped, 2 total`。

![](https://cdn-images-1.medium.com/max/800/1*Lwwg-6KHJz5hJhnh7xfeoA.png)

当你捅咕半天编辑器，然后实现了这两个 API 之后，再次运行这条命令，会返回成功提示。

![](https://cdn-images-1.medium.com/max/800/1*tfq7p-06V2Pzyn-7r5WKJg.png)


### 有啥用？？？

你可以先跟客户端开发的兄弟，商议好 API 接口和返回值，然后用 apiary.io 提供的 mock server 让客户端跟服务端同步开发，而不用等你。

## 相关

* [There Are Four API Design Editors To Choose From Now](https://apievangelist.com/2014/11/21/there-are-four-api-design-editors-to-choose-from-now/)
* [Quickly Prototype APIs with Apiary](https://sendgrid.com/blog/quickly-prototype-apis-apiary/)
* [API Blueprint](https://apiblueprint.org/)
* [API Blueprint Tutorial](https://apiblueprint.org/documentation/tutorial.html)
* [API Blueprint examples](https://github.com/apiaryio/api-blueprint/tree/master/examples)
* [Writing Dredd Hooks In Ruby](https://dredd.readthedocs.io/en/latest/hooks-ruby/)
