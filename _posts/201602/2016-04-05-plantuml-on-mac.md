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
brew install plantuml
apm install language-plantuml
apm install plantuml-viewer
```

### Configure the plugins

```
$ which dot
/usr/local/bin/dot
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

## 颜色配置

为了让颜色与众不同，我们还可以自定义一些显示相关的配置。

详细的外观参数在 [这个文档](http://plantuml.com/zh/skinparam)。想调教一份自己心仪的配色可不容易，下面列一下我觉得不错的：

* https://amasuda.xyz/post/2018-06-01-plantuml-style/
* [plantuml-style-c4](https://github.com/xuanye/plantuml-style-c4)
* [plantuml-styles](https://github.com/inthepocket/plantuml-styles) 我推荐这套
* [我魔改的](https://gist.github.com/wjp2013/339d54b66ff8a9cafdc9a9463028179b)

```
@startuml
!include https://gist.githubusercontent.com/wjp2013/339d54b66ff8a9cafdc9a9463028179b/raw/9feb53e77b64bfb339eb23b72ec8bce84bcb87fd/style.plantuml

actor Client

Client -> Java : Request
activate Java


Java -> CircleService : Request Tip tax and Tip allow
CircleService --> Java

alt 圈子关闭打赏
Java --> Client : Response error msg
else
Java -> TopicService : 我是否可打赏该对象
TopicService --> Java

alt 不可继续打赏
Java --> Client : Response error msg
else

Java -> WalletService : 打赏者扣钱，创作者加钱
WalletService --> Java

opt 圈主需要抽成
Java -> WalletService : 圈主加钱
WalletService --> Java
end

Java -> TopicService : 创建打赏记录
TopicService --> Java

Java -> TransactionService : 创建打赏者和创作者的复式账单
TransactionService --> Java

opt 圈主需要抽成
Java -> TransactionService : 创建圈主抽成的单式账单
TransactionService --> Java
end

Java --> Client : Response body

end
end

deactivate Java

@enduml
```

![](https://raw.githubusercontent.com/wjp2013/wjp2013.github.io/master/assets/images/pictures/2016-04-05-plantuml-on-mac/01.png)

## 相关链接

* [官方中文文档](http://translate.plantuml.com/zh)
* [PLANTUML IN ATOM](http://trevershick.github.io/atom/2015/12/04/plantuml-snippets.html)
* [使用 Sublime + PlantUML 高效地画图](http://www.jianshu.com/p/e92a52770832)
* [在 Mac 上使用 PlantUML 高效画图](http://blog.yourtion.com/use-plantuml-on-mac.html)
* [PlantUML画复杂流程图](https://blog.csdn.net/zhangjikuan/article/details/53484558)
