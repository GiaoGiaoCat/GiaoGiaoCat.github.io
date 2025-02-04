---
layout: post
title:  "Cursor 增强上下文管理"
date:   2025-02-03 19:00:00

categories: tool
tags: ai
author: "Victor"
---

## Summarized Composers

可以总结 Composer 历史对话。

1. 当 Composer 对话有一定历史记录时，可以选择 `@Summarized Composers` 总结对话。
2. 开启一个新的 Composer 对话后，再次输入 `@Summarized Composers` 可以选择历史对话，并且可以选择多个历史对话进行汇总。

## Optional Long Context

允许 Cursor 在 Composer 模式下使用更长的代码上下文。默认情况下该功能关闭，开启后会消耗更多的 fast requests 次数。

由于该功能目前尚未完全完善，建议仅在进行复杂的代码重构时启用此功能。

## README.md 记录文件项目结构

记录项目的整体目标、功能模板、每个模块的用途、项目所使用的技术栈和依赖库，以及关键节点的变更和开发进度。在开发过程中，可以让 Cursor 保持 README 文件的更新，每次完成一个模块后就在 README 中记录对应模块的信息。

这个方案与 `@Summarized Composers` 类似。虽然 `@Summarized Composers` 使用起来简单，但如果 Composer 中有大量的调试和 debug 信息，生成的有效信息会减少。

## Notepads

Notepads 是 Cursor 的上下文共享工具，弥补了 Composer 和 Chat 在上下文管理方面的不足。适合存放需要频繁引入、不太变化的、可重复使用的代码和模板，如各种 API 调用示例等。

1. 动态模板生成
  * 创建常见代码模式的模板
  * 存储特定项目的脚手架规则
  * 保持团队代码结构的一致性
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
