---
layout: post
title: "Claude Code 安装与使用教程"
subtitle: "Anthropic 官方出品的终端 AI 编程助手"
date: 2026-05-16 10:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-halting.jpg"
catalog: true
tags:
  - 教程
  - AI 工具
  - Claude
  - 效率
---

## 什么是 Claude Code

Claude Code 是 Anthropic 官方推出的终端 AI 编程助手（CLI 工具），可以在命令行中直接与 Claude 对话，让它帮你阅读代码、修改文件、运行命令、搜索代码库——全程不需要离开终端。

## 安装

### 前置条件

- Node.js 18.0 或更高版本
- 一个 Anthropic API Key（[在此获取](https://console.anthropic.com/)）
- Git（用于版本追踪和回退）

### 安装步骤

```bash
# 全局安装
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version
```

### 配置 API Key

```bash
# 方式一：环境变量（推荐）
export ANTHROPIC_API_KEY="sk-ant-xxx"

# 方式二：首次运行时交互式配置
claude
# 首次运行会引导你输入 API Key
```

```powershell
# Windows PowerShell 设置
$env:ANTHROPIC_API_KEY = "sk-ant-xxx"
```

建议将 API Key 写入 shell 配置文件（`.bashrc` / `.zshrc` / `$PROFILE`），避免每次手动设置。

## 基本使用

### 启动 Claude Code

```bash
# 在项目目录中启动
cd my-project
claude

# 或指定目录
claude /path/to/project
```

### 日常操作示例

```bash
# 代码理解
"这个项目的入口文件在哪里？帮我梳理一下整体架构"

# 代码修改
"把 utils.js 里的 fetchData 函数改成 async/await 风格"

# 创建新文件
"帮我创建一个 Express 服务器，包含 /api/users 路由"

# Debug
"为什么 npm start 报错？帮我排查一下"

# Git 操作
"帮我写一个清晰的 commit message 来描述最近的改动"

# 测试
"给 src/auth.js 写单元测试"
```

## 核心功能

### 1. 文件操作

Claude Code 可以直接读写你的项目文件：

- ✅ 创建新文件
- ✅ 修改现有文件（精确到行）
- ✅ 删除文件
- ✅ 跨文件重构

每项文件操作都会**先展示 diff**，让你确认后再应用，安全可控。

### 2. 终端命令执行

```bash
# Claude Code 可以帮你运行命令
"帮我安装项目依赖"
→ npm install

"看看最近的 git 提交"
→ git log --oneline -10

"运行测试并告诉我结果"
→ npm test
```

**安全机制**：涉及文件删除、强制推送等危险操作会请求确认。

### 3. 代码库理解

Claude Code 会索引你的项目，理解代码结构：

- 自动发现项目结构和入口文件
- 理解模块间依赖关系
- 追踪函数调用链
- 跨文件搜索引用

### 4. Git 集成

```bash
"总结一下上次提交以来的所有改动"
"把我的改动分成两个干净的 commit"
"帮我解决合并冲突"
"生成一个 PR 描述"
```

## 进阶技巧

### 自定义配置

在项目根目录创建 `.claude/settings.json`：

```json
{
  "permissions": {
    "allow": [
      "npm test",
      "npm run lint",
      "git diff",
      "git log"
    ],
    "deny": [
      "rm -rf",
      "git push --force"
    ]
  }
}
```

### 使用 CLAUDE.md

在项目根目录放一个 `CLAUDE.md` 文件，告诉 Claude Code 项目的约定和上下文：

```markdown
# 项目约定

- 使用 ESLint + Prettier 格式化代码
- 测试框架是 Jest
- 组件文件用 PascalCase
- API 路由统一放在 /api 目录下
```

### 选择模型

```bash
# 使用 Opus（最强，适合复杂任务）
claude --model claude-3-opus-20240229

# 使用 Sonnet（平衡，日常使用推荐）
claude --model claude-3-sonnet-20240229

# 使用 Haiku（最快，简单任务）
claude --model claude-3-haiku-20240307
```

### 会话管理

```bash
# 恢复上次会话
claude --resume

# 查看会话历史
claude --history
```

## Claude Code vs 其他工具

| 特性 | Claude Code | GitHub Copilot | Cursor |
|------|------------|----------------|--------|
| 运行方式 | 终端 CLI | IDE 插件 | IDE (独立编辑器) |
| 文件操作 | ✅ 直接读写 | ❌ 仅建议 | ✅ 直接修改 |
| 终端命令 | ✅ 可执行 | ❌ | ✅ 可执行 |
| 代码理解 | ✅ 索引全项目 | ✅ 上下文感知 | ✅ 索引全项目 |
| 开源 | ❌ 闭源 | ❌ 闭源 | ❌ 闭源 |
| 定价 | API 按量付费 | $10/月起 | $20/月起 |

## 常见问题

### Q: 如何控制成本？

- 简单任务用 Haiku（极便宜）
- 复杂任务用 Sonnet（性价比最高）
- 避免让 Claude 读取不必要的超大文件
- 善用 `.claude/settings.json` 限制操作范围

### Q: Claude Code 能看到我的代码吗？

代码通过 Anthropic API 传输，Anthropic 声明不会用 API 数据训练模型。敏感项目建议评估后再使用。

### Q: 支持哪些编程语言？

几乎所有。Claude 的训练数据覆盖了主流的编程语言，Python、JavaScript、TypeScript、Java、Go、Rust 等表现尤其出色。

## 总结

Claude Code 适合喜欢在终端工作、希望 AI 能直接操作文件和执行命令的开发者。它的核心优势是：

- 🎯 **Agentic**：不只是建议，能真正帮你做事
- 🔒 **安全**：操作前展示 diff，危险命令需确认
- 🏗 **项目感知**：理解整个代码库，不只是当前文件

> 官方文档：[Anthropic Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code)
