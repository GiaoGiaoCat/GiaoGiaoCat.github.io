---
layout: post
title:  "使用 Typora 画图"
date:   2020-11-17 18:10:00

categories: tool
tags: uml
author: "Victor"
---

## 流程图基础

流程图，顾名思义，就是表示一个事件或活动的流程的图示。流程图的符号有很多，但常用的也就几个。

* 圆角矩形：表示开始和结束
* 矩形：表示过程，也就是整个流程中的一个环节
* 单向箭头线段：表示流程进行方向
* 菱形：表示判断、决策
* 圆形：表示连接。为避免流程过长或有交叉，可将流程切开，圆形即相当于切口处的连接头（成对出现）
* 另外还有嵌入在以上符号中的描述文本

![菱形](https://pic1.zhimg.com/80/v2-1b484aecaf693db962335b20f50747d0_1440w.jpg)
![圆形](https://pic2.zhimg.com/80/v2-9ea686ae562c50db76d7c823fde885fd_1440w.jpg)

## Mermaid

一种简单的类似 Markdown 的脚本语言，通过 JavaScript 编程语言，将文本转换为图片。因此，真正实现画图功能的并不是 Typora 本身，它只是内置了对 Mermaid 的支持。

![](https://pic2.zhimg.com/80/v2-4e44a08fa37bdb6b03df9fcec8480ed9_1440w.jpg)

Mermaid 支持绘制非常多种类的图，常见的有时序图、流程图、类图、甘特图等等。先在 Typora 中，输入 ```mermaid 然后敲击回车，即可初始化一张空白图。

### 流程图

语法解释：graph 关键字就是声明一张流程图，TD 表示的是方向，这里的含义是 Top-Down 由上至下。

```
graph TD;
    A-->B;
    A-->C;
    B-->D;
```

![](https://pic4.zhimg.com/80/v2-d4e7402e1dce5cefb924776e01b0bffb_1440w.jpg)

#### 节点的画法

```
graph TB
    A(开始)
    B[打开冰箱门]
    C{"冰箱小不小？"}
    D((连接))
```

![](https://pic2.zhimg.com/80/v2-54708b0d67df0d37cfec51823bbe2531_1440w.jpg)

#### 线段的画法

```
graph TB
    A[把大象放进去] --> B{"冰箱小不小？"}
    B -->|不小| C[把冰箱门关上]
    B -->|小| D[换个大冰箱]
```

![](https://pic2.zhimg.com/80/v2-19685edec442c83299bd291ebcf255e9_1440w.jpg)


#### 完整例子

```
graph TB
    Start(开始) --> Open[打开冰箱门]
    Open --> Put[把大象放进去]
    Put[把大象放进去] --> IsFit{"冰箱小不小？"}

    IsFit -->|不小| Close[把冰箱门关上]
    Close --> End(结束)

    IsFit -->|小| Change[换个大冰箱]
    Change --> Open
```

![](https://pic3.zhimg.com/80/v2-e958143a8141c22f1c1f71579831f77a_1440w.jpg)

### 时序图

语法解释：->> 代表实线箭头，-->> 则代表虚线。

```
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John-->>Alice: Great!
```

![](https://pic3.zhimg.com/80/v2-decde80f8f46a4a5399f20c55bb4b00a_1440w.jpg)

```
sequenceDiagram
    小程序 ->> 小程序 : wx.login()获取code
    小程序 ->> + 服务器 : wx.request()发送code
    服务器 ->> + 微信服务器 : code+appid+secret
    微信服务器 -->> - 服务器 : openid
    服务器 ->> 服务器 : 根据openid确定用户并生成token
    服务器 -->> - 小程序 : token
```

![](https://pic2.zhimg.com/80/v2-5fc5a332a3d7bbc58f87e19463963739_1440w.jpg)

### 其它

* 状态图
* 类图
* 甘特图
* 饼图

### 导出

绘制好的图片可以选择菜单/文件/导出，导出为图片或者网页格式。在网页中图片是以 SVG 格式渲染的，你可以复制 SVG 内容，导入到 SVG 的图片编辑器中进一步操作。

Mermaid 官方有一个[在线的工具](https://mermaid-js.github.io/mermaid-live-editor/)，可以导出 SVG 和 PNG。

## 相关阅读

* [Mermaid](https://mermaid-js.github.io/mermaid/#/)
* [使用 Typora 画图（类图、流程图、时序图）](https://zhuanlan.zhihu.com/p/172635547)
