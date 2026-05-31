---
layout: post
title: "毕业设计：基于YOLO的燃料电池缺陷检测系统"
subtitle: "深度学习 + 工业缺陷检测的完整实践"
date: 2026-05-20 12:00:00
author: "TheShycute"
header-img: "img/post-bg-dreamer.jpg"
catalog: true
tags:
  - 项目
  - 深度学习
  - YOLO
  - 毕业设计
  - Python
---

## 项目背景

燃料电池作为清洁能源的重要载体，其组件装配质量直接影响性能和安全性。传统的人工目检效率低、易疲劳、一致性差。本文介绍一套基于深度学习的自动化缺陷检测方案。

## 系统功能

本系统基于 **YOLO 实例分割模型**，能够自动识别燃料电池组件的排列顺序，并检测以下装配缺陷：

- 🔴 **漏装** — 组件缺失
- 🟡 **多装** — 多余组件
- 🟠 **错装** — 组件位置错误
- 🔵 **倒装** — 组件方向错误

## 技术栈

| 层级 | 技术选型 |
|------|----------|
| 前端 | React 18 + TypeScript + Tailwind CSS + Framer Motion + Zustand |
| 后端 | FastAPI (Python) + MySQL |
| 模型 | YOLO 实例分割 |
| 部署 | Docker |

## 项目亮点

- **全栈开发**：前后端分离架构，RESTful API 设计
- **现代化前端**：React 18 + TypeScript 保证类型安全，Zustand 轻量状态管理
- **高性能推理**：FastAPI 异步框架配合 YOLO 模型，实现实时检测
- **优雅 UI**：Tailwind CSS + Framer Motion 实现流畅的动画交互

## 总结

这个项目是我本科阶段的毕业设计，从需求分析、技术选型到最终部署，全程独立完成。通过这个项目，我深入理解了深度学习在工业检测领域的应用场景和工程实践。

> 项目仓库：[Gitee - mr-bs](https://gitee.com/TheShycute/mr-bs)（私有）
