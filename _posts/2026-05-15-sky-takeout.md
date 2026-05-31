---
layout: post
title: "苍穹外卖——前后端分离的餐饮外卖系统"
subtitle: "Spring Boot + Vue 全栈项目实战"
date: 2026-05-15 12:00:00
author: "TheShycute"
header-img: "img/post-bg-alibaba.jpg"
catalog: true
tags:
  - 项目
  - Java
  - Spring Boot
  - Vue
  - 全栈
---

## 项目介绍

苍穹外卖是一个完整的外卖点餐系统，涵盖用户端、商家端和管理后台三大模块，是学习前后端分离架构的绝佳实战项目。

## 功能模块

### 用户端
- 🛒 菜品浏览与搜索
- 📋 购物车管理
- 💳 在线下单与支付
- 📦 订单状态追踪

### 商家端
- 📊 营业数据概览
- 🍳 菜品管理（CRUD）
- 📋 订单处理
- 📈 销售统计

### 管理后台
- 👥 用户管理
- 🏪 商家审核
- 📊 数据报表

## 技术栈

```
后端: Spring Boot + MyBatis + MySQL + Redis
前端: Vue.js + Element UI
认证: JWT Token
部署: Docker + Nginx
```

## 核心亮点

- **RESTful API 设计**：清晰的接口规范，前后端分离
- **JWT 无状态认证**：支持多角色（用户/商家/管理员）权限控制
- **缓存优化**：Redis 缓存热门菜品数据，提升响应速度
- **接口文档**：Swagger 自动生成 API 文档

> 项目仓库：[Gitee - sky-take-out](https://gitee.com/TheShycute/sky-take-out)（私有）
