---
layout: post
title: "Git 高效工作流与进阶技巧"
subtitle: "从 add/commit/push 到真正驾驭版本控制"
date: 2026-05-28 10:00:00
author: "TheShycute"
header-img: "img/post-bg-digital-native.jpg"
catalog: true
tags:
  - 教程
  - Git
  - 工具
  - 效率
---

## Git 不止是 add/commit/push

很多开发者对 Git 的使用停留在最基础的三条命令。但实际上，Git 有非常多强大的功能可以大幅提升工作效率。

## 分支管理最佳实践

### Git Flow 工作流

```
main     ●────────────●──────────●  (稳定发布)
          \          /
develop    ●──●──●──●──●──●──●    (开发主线)
              \    /
feature        ●──●                (功能分支)
```

### 常用分支命令

```bash
# 创建并切换到新分支
git checkout -b feature/new-feature

# 合并分支
git checkout develop
git merge feature/new-feature

# 删除已合并的分支
git branch -d feature/new-feature
```

## 进阶技巧

### 1. 交互式暂存（部分提交）

```bash
git add -p
# 逐个选择要暂存的代码块
```

### 2. 修改最后一次提交

```bash
# 修改 commit 信息
git commit --amend -m "新的提交信息"

# 补充遗漏的文件
git add forgotten-file.py
git commit --amend --no-edit
```

### 3. 暂存工作现场

```bash
# 暂存当前修改
git stash

# 查看暂存列表
git stash list

# 恢复暂存
git stash pop
```

### 4. 优雅的回退

```bash
# 撤销工作区修改
git checkout -- filename

# 撤销暂存区
git reset HEAD filename

# 回退到某个提交（保留修改）
git reset --soft HEAD~1

# 完全回退
git reset --hard HEAD~1
```

### 5. Cherry-pick（摘樱桃）

```bash
# 只将某个特定提交合并到当前分支
git cherry-pick <commit-hash>
```

## 提交信息规范

一个好的 commit message：

```
<type>: <简短描述>

<详细说明（可选）>

类型：feat / fix / docs / style / refactor / test / chore
```

例子：

```
feat: 添加用户登录功能

- 实现 JWT token 认证
- 添加登录/注册 API
- 编写单元测试
```

## .gitignore 最佳实践

```gitignore
# Python
__pycache__/
*.py[cod]
venv/

# Node
node_modules/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db
```

## 总结

Git 是一个极其强大的工具，花点时间深入掌握它能让你在日常开发中事半功倍。建议从 `git log`、`git reflog`、`git bisect` 等命令开始探索。

> 相关文章：[GitHub Pages 博客搭建指南](https://theshycute.github.io/2026/05/31/github-pages-blog/)
