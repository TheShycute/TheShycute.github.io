---
layout: post
title: "Git & GitHub 快速上手指南"
subtitle: "从零开始掌握版本控制"
date: 2026-05-31 14:00:00
author: "TheShycute"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
  - 技术
  - Git
  - GitHub
---

## 什么是 Git？

Git 是目前最流行的分布式版本控制系统，由 Linus Torvalds 在 2005 年创建。它帮助开发者追踪代码的每一次变动。

## 基本命令

```bash
# 初始化仓库
git init

# 克隆远程仓库
git clone <url>

# 查看状态
git status

# 添加文件到暂存区
git add .

# 提交更改
git commit -m "提交信息"

# 推送到远程仓库
git push origin main
```

## GitHub Pages

GitHub Pages 是 GitHub 提供的免费静态网站托管服务。只需要创建一个名为 `<username>.github.io` 的仓库，推送 HTML/CSS/JS 文件，就可以拥有一个免费的个人网站。

### 部署步骤

1. 创建仓库 `你的用户名.github.io`
2. 上传网站文件
3. 等待几分钟，访问 `https://你的用户名.github.io`

## 持续学习

掌握 Git 和 GitHub 是现代开发者的必备技能。多用多练，慢慢就会熟练起来！
