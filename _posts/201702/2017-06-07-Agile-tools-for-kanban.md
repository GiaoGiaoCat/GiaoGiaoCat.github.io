---
layout: post
title:  "Agile tools for kanban"
date:   2017-06-07 11:30:00
categories: other
tags: scrum
author: "Victor"
---

## 项目管理

### 项目管理的价值

1.对企业而言：降低项目的耗时，节约企业资源，为企业效益作出正向贡献。
2.对管理者而言：掌握项目进程。项目结束后还能沉淀企业数字资产，为今后的优化升级提供数据支持和经验模板。
3.对员工而言：帮助员工把握工作重点，让项目协作权责分明、分工清晰、流程有条不紊；有利于员工个人能力的发挥，学到更多东西，增强团队凝聚力。

### 成功项目管理的关键要素

* 优秀的项目团队
* 清晰的项目流程
* 高效地沟通协作
* 分工明确责任到人
* 严格把控项目质量
* 企业数字资产沉淀

## Kanban

Kanban 方式是基于持续发布的想法。通过 board 的 columns 和 lanes 的状态来跟踪工作进度。kanban 通过 4 个支柱来帮助团队发布产品：

* 持续发布 continuous releases
* 限制进行中的任务 WIP(work in progress)
* 工作列表 the list of work
* columns 或 lanes

它的好处有：

* 流程可视化：把工作拆分任务卡片，显示每件任务在流程中处于什么位置。
* 度量生产周期：对流程进行调优，尽可能缩短生产周期，并使其可预测。


## Features for kanban

看板的一个好处是，你的团队几乎没有学习成本。JIRA 默认就提供相关的工作流程，让你方便的创建 kanban board，添加一些 issues 或 user stories。一旦你的团队习惯了 board，你可以开始自定义项目、工作流程、issue 类型，以满足团队的需求。下面介绍一些 JIRA 提供的功能。

### Story cards

Board 将显示每个 story、issue、bug 或 task 中的相关信息。点击它们可以查看所有细节，包括相关源码和 pull requests、优先级，注释，附件等。

### WIP limit configuration

限制每个状态的用户故事的数量。它对于防止某些特别的状态称为项目中的瓶颈，确保整个工作流程的顺利十分重要。

### Swimlanes and columns

配置列以表示工作流的状态。例如 “To Do”，“In Progress” 和 “Done”。 Add swimlanes to group work into streams by epics, assignees, or projects or whatever makes sense for your team.

### Flexible workflows

可以根据需要定义和配置不同的工作流和 issue 类型，或者所有 board 使用相同的 issue 类型。你想咋搞就咋搞。

### Kanban board

Kanban board 让团队知道下一步干啥。一个工作（item or card）完成后，团队可以立刻搞下一个。

## 相关链接

* [Worktile产品百科 | 项目管理](https://worktile.com/blog/features/1e93f8d85fae6a)
