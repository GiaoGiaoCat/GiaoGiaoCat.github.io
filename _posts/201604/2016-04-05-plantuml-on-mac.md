---
layout: post
title:  "在 Mac 上使用 PlantUML 画 UML"
date:   2016-04-04 20:00:00
categories: tool
tags: uml
author: "Victor"
---

## 什么是 PlantUML

PlantUML 是一个画图脚本语言，用它可以快速地画出各种 UML 图，时序图、流程图、用例图、状态图、组件图什么的。

## 安装
我最近一年主力编辑器是 Atom，所以下面的演示方案也是基于 Atom

```
brew install graphviz
apm install language-plantuml
apm install plantuml-viewer
```

## 简单使用
使用的话比较简单，绘图的内容需要包含在 `@startuml` 和 `@enduml` 中，不然会报错。

```
@startuml
Bob -> Alice : Hello, how are you
Alice -> Bob : Fine, thank you, and you?
@enduml
```

## 复杂点的

```
@startuml

start
:"步骤1处理";
:"步骤2处理";
if ("条件1判断") then (true)
    :条件1成立时执行的动作;
    if ("分支条件2判断") then (no)
        :"条件2不成立时执行的动作";
    else
        if ("条件3判断") then (yes)
            :"条件3成立时的动作";
        else (no)
            :"条件3不成立时的动作";
        endif
    endif
    :"顺序步骤3处理";
endif

if ("条件4判断") then (yes)
:"条件4成立的动作";
else
    if ("条件5判断") then (yes)
        :"条件5成立时的动作";
    else (no)
        :"条件5不成立时的动作";
    endif
endif
stop

@enduml
```

![](http://blog.yourtion.com/images/2015/12/palntuml-demo1.png)

## 相关链接

* [官方中文文档](http://translate.plantuml.com/zh)
* [使用 Sublime + PlantUML 高效地画图](http://www.jianshu.com/p/e92a52770832)
* [在 Mac 上使用 PlantUML 高效画图](http://blog.yourtion.com/use-plantuml-on-mac.html)
* [PlantUML画复杂流程图](https://blog.csdn.net/zhangjikuan/article/details/53484558)
