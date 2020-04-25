---
layout: post
title:  "替代 Dash"
date:   2020-04-24 17:10:00

categories: tool
tags: mac
author: "Victor"
---

## Dash

从知道 Dash 就开始付费。后面不停的迭代版本，然后不停的收钱。主要是版本升级了除了 UI 改改，其实功能没什么大的变化。再连续付费 3 次之后，终于决定再见了。

## 替代品

Dash 有两个免费的替代品。

1. Alfred Workflow：[Dev Doctor](http://wemakeawesomesh.it/alfred-dev-doctor/) ，实时查询每个语言的线上手册网站
2. 另外一个是 devdocs.io，也有对应的 [Alfred Workflow](https://github.com/yannickglt/alfred-devdocs)，因为支持本地 offline 所以响应速度稍快。

不过两者的快捷键完全一样，只能留其一。

### 用法

首先需要添加一个文档。

```
cdoc:add javascript
```

其它的配置命令。

```
cdoc:list: 列出所有可添加的文档，第一次执行会有点慢稍等一会
cdoc:add: 添加文档
cdoc:remove: 删除文档
cdoc:all: 添加所有文档，不推荐使用
cdoc:nuke: 删除所有已添加的文档
cdoc:refresh: 刷新所有文档的缓存
cdoc:alias: 为文档创建别名
cdoc:unalias: 删除文档别名
```

### 对比

缺点是 preview 有点慢，没办法国内网速就这样。所以需要打开浏览器才能看到详细文档。
有点是免费。
