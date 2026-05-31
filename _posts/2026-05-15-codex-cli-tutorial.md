---
layout: post
title: "Codex CLI 安装与使用教程"
subtitle: "OpenAI 开源终端 AI 编程助手，功能强大且完全免费"
date: 2026-05-15 10:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
catalog: true
tags:
  - 教程
  - AI 工具
  - Codex
  - OpenAI
  - 开源
---

## 什么是 Codex CLI

Codex CLI 是 OpenAI 开源的终端 AI 编程助手。与 Claude Code 类似，它可以在终端中帮你**读代码、改文件、跑命令、搜索项目**——而且是**完全开源免费的**。

> 🔗 GitHub：[openai/codex](https://github.com/openai/codex)

## 安装

### 前置条件

- Node.js 18.0+
- OpenAI API Key（[在此获取](https://platform.openai.com/api-keys)）
- Git

### 方式一：npm 全局安装（推荐）

```bash
npm install -g @openai/codex
codex --version
```

### 方式二：桌面应用

Codex 也提供 Windows / macOS 桌面应用，内置终端和 Codex CLI：

1. 前往 [GitHub Releases](https://github.com/openai/codex/releases) 下载安装包
2. 安装后在应用中直接使用 `codex` 命令

### 配置 API Key

```bash
# Linux / macOS
export OPENAI_API_KEY="sk-xxx"

# Windows PowerShell
$env:OPENAI_API_KEY = "sk-xxx"
```

首次运行时也会引导你配置。

## 基本使用

### 启动 Codex

```bash
# 在项目目录中启动
cd my-project
codex

# 交互模式下直接提问
codex "帮我解释这个项目的结构"
```

### 日常场景

```bash
# 阅读代码
"src/ 目录下的模块是怎么组织的？画个流程图"

# 修改代码
"把 user.service.ts 里的密码存储改成 bcrypt 加密"

# 创建项目
"帮我创建一个 React + TypeScript 项目，带路由和状态管理"

# 修 Bug
"登录接口返回 500，帮我排查"

# 写测试
"给 order.service.ts 写单元测试，用 Jest"

# 运行命令
"运行 npm test，告诉我哪些测试没通过"
```

## 核心功能

### 1. 代码理解与编辑

Codex CLI 可以直接读取和修改文件：

- 📖 读取项目文件，理解代码逻辑
- ✏️ 精准编辑（通过 `apply_patch` 工具修改到行级）
- 🔍 搜索代码库（调用 `rg` 等工具）
- 📊 分析项目结构

### 2. 终端命令执行

```bash
# Codex 可以帮你执行各种命令
"安装项目依赖"         → npm install
"查看最近的 git 记录"   → git log
"启动开发服务器"        → npm run dev
"运行所有测试"          → npm test
```

**安全机制**：可配置为需要用户确认后才执行命令。

### 3. 计划模式（Plan）

Codex CLI 支持先制定计划，再逐步执行：

```
用户：帮我做一个用户管理系统

Codex：
📋 计划：
1. 创建 User 数据模型
2. 实现 CRUD API
3. 添加 JWT 认证中间件
4. 创建前端页面
5. 编写测试
是否按此计划进行？
```

你可以审查计划后再让它执行，每一步都会展示变更。

### 4. Skills（技能系统）

Codex 有丰富的 Skills 生态，可以直接调用：

| Skill | 功能 |
|-------|------|
| `browser` | 浏览器自动化测试 |
| `pdf` | PDF 读取与创建 |
| `pptx` | PPT 生成 |
| `xlsx` | Excel 操作 |
| `frontend-design` | 前端页面设计 |
| `imagegen` | AI 图片生成 |

```bash
# 使用 skill
codex "用 frontend-design skill 帮我做一个 landing page"
```

### 5. MCP 集成

Codex 支持 MCP（Model Context Protocol）——可以连接外部服务和数据库：

```json
// .codex/mcp.json
{
  "servers": [
    {
      "name": "my-database",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/mydb"
      }
    }
  ]
}
```

连接后可以直接让 Codex 查询数据库、分析数据。

## 进阶配置

### 项目级配置 `.codex/config.toml`

```toml
[model]
# 选择模型
name = "gpt-4o"
# 或使用 o1/o3 进行深度推理
# name = "o3"

[permissions]
# 命令白名单
allow = ["npm test", "npm run lint", "git diff"]
# 命令黑名单
deny = ["rm -rf", "git push --force"]

[sandbox]
# 启用文件沙箱
enabled = true
```

### 项目指南 `AGENTS.md`

在项目根目录创建 `AGENTS.md` 告诉 Codex 项目约定：

```markdown
# 项目编码规范

- 使用 TypeScript 严格模式
- 测试覆盖率 > 80%
- 组件文件命名：PascalCase.tsx
- API 路由：/api/v1/*
- 禁止使用 any 类型
```

## Codex vs 其他工具

| 特性 | Codex CLI | Claude Code | Cursor | Copilot |
|------|----------|-------------|--------|---------|
| 开源 | ✅ MIT | ❌ | ❌ | ❌ |
| 终端原生 | ✅ | ✅ | ❌ | ❌ |
| 文件编辑 | ✅ 精确到行 | ✅ | ✅ | ❌ 仅建议 |
| 命令执行 | ✅ | ✅ | ✅ | ❌ |
| Skills 生态 | ✅ 丰富 | ❌ | ❌ | ❌ |
| MCP 集成 | ✅ | ✅ | ❌ | ❌ |
| 成本 | API 费用 | API 费用 | $20/月 | $10/月 |

## 常见问题

### Q: Codex CLI 安全吗？

开源意味着代码可审查。沙箱机制可以限制文件访问范围，权限控制可以管理命令执行。建议启用 `sandbox.enabled = true`。

### Q: 需要什么 API Key？

需要 OpenAI API Key。支持 GPT-4o、o1、o3 等模型。未来可能支持更多模型提供商。

### Q: 和 Claude Code 怎么选？

- 追求**开源透明** → Codex CLI
- 需要**最强编码能力** → Claude Code (Opus/Sonnet)
- 两者都试试，看哪个更顺手！

## 总结

Codex CLI 的核心优势：

- 🔓 **完全开源**：MIT 协议，代码公开可审计
- 🆓 **免费使用**：只需 API 费用，无订阅
- 🧩 **Skills 生态**：丰富的内置技能
- 🔌 **MCP 协议**：可连接数据库、API 等外部服务
- 🏖 **沙箱安全**：文件操作有边界控制
- 📋 **Plan 模式**：先计划再执行，透明可控

> 官方仓库：[github.com/openai/codex](https://github.com/openai/codex)
