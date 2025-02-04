---
layout: post
title:  "cursor 增强上下文管理"
date:   2025-02-03 19:00:00

categories: tool
tags: ai
author: "Victor"
---

## Summarized Composers

可以总结 Composer 历史对话。

1. 当 Composer 对话有一定历史记录时，可以选择 `@Summarized Composers` 总结对话。
2. 开启一个新的 Composer 对话，再次输入 `@Summarized Composers` 可以选择历史对话，并可以选择多个历史对话。

## Optional Long Context

允许 Cursor 在 COmposer 模式下使用更长的代码上下文。默认关闭，开启之后会消耗更多的 fast requests 次数。

因为目前功能还不完善，建议只有在进行复杂的代码重构时，再去设置中开启这个功能。

## README.md 记录文件项目结构

记录项目的整体目标，功能模板和每个模块的用途，项目所使用的技术栈和依赖库，以及关键节点的变更和开发进度。在开发过程中，可以让 Cursor 保持 README 文件的更新，每次完成一个模块后就在 README 中记录对应模块的信息。

这个方案和 Summarized Composers 很像。虽然 `@Summarized Composers` 用起来简单，但若 Composer 中出现大量的 debug 和调试信息，其 Summarized Composers 产生的有效信息也更少。

## Notepads

是 Cursor 的上下文共享工具，用来弥补 Composer 和 Chat 在上下文管理能力的不足。适合写一些需要频繁引入，不太变化的，可重复使用的代码和模板，比如各种 API 的调用示例等。

1. 动态模板生成
  * 创建常见的代码模式的模板
  * 存储特定项目的脚手架规则
  * 保持团队代码结构一致
2. 架构文档
  * 前端规范
  * 后端设计模式
  * 数据模型文档
  * 系统架构指南
3. 开发指南
  * 编码规范
  * 项目特定规则
  * 最佳实践
  * 团队规范
