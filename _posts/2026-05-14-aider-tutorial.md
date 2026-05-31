---
layout: post
title: "Aider 安装与使用教程"
subtitle: "终端里的 AI 结对编程搭档，支持多模型、强 Git 集成"
date: 2026-05-14 10:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
  - 教程
  - AI 工具
  - Aider
  - 开源
  - 效率
---

## 什么是 Aider

Aider 是一个开源的终端 AI 编程助手，核心理念是 **"AI 结对编程"**——你写需求，它写代码，在同一个 Git 仓库中协作。

和 Claude Code、Codex CLI 最大的不同在于，Aider 的 Git 集成深入到骨子里——每次修改都是一个干净的 commit，你可以像审查同事的代码一样审查 AI 的改动。

> 🔗 GitHub：[paul-gauthier/aider](https://github.com/paul-gauthier/aider)

## 安装

### 前置条件

- Python 3.8+
- Git
- 任意 LLM 的 API Key（OpenAI / Anthropic / 本地模型）

### 安装步骤

```bash
# pip 安装
pip install aider-chat

# 或使用 pipx（推荐，隔离环境）
pipx install aider-chat

# 验证
aider --version
```

### 配置 API Key

Aider 支持几乎所有主流 LLM：

```bash
# OpenAI
export OPENAI_API_KEY="sk-xxx"

# Anthropic (Claude)
export ANTHROPIC_API_KEY="sk-ant-xxx"

# DeepSeek
export DEEPSEEK_API_KEY="sk-xxx"

# 通义千问（国内推荐）
export DASHSCOPE_API_KEY="sk-xxx"
```

## 基本使用

### 启动 Aider

```bash
# 在项目目录中启动，指定初始文件
cd my-project
aider src/main.py

# 启动时就告诉它要做什么
aider --message "帮我重构这个文件，用 async/await 替代回调"

# 空目录创建新项目
aider --message "创建一个 Flask 博客应用"
```

### 工作流程

```
① 你描述需求 → "添加用户登录功能"
② Aider 读取代码 → 理解上下文
③ Aider 修改文件 → 展示 diff
④ 你审查 diff → 确认或拒绝
⑤ Aider 自动 commit → Git 历史清晰
```

### 日常场景

```bash
# 添加新功能
aider src/app.py
"添加一个 /api/health 健康检查接口"

# 重构代码
aider src/
"把所有的 print 语句替换成 logging 模块"

# 写测试
aider tests/
"给 user_service.py 写完整的单元测试"

# 修 Bug
aider src/utils.py
"这个函数在输入为空列表时会报错，帮我修复"

# 代码审查
aider src/
"review 一下最近的改动，有什么可以改进的地方"
```

## 核心功能

### 1. 多模型支持

Aider 最大的优势是**不绑定任何模型提供商**：

```bash
# 使用 Claude Sonnet（日常推荐）
aider --model claude-3-5-sonnet-20241022

# 使用 GPT-4o
aider --model gpt-4o

# 使用 DeepSeek V3（性价比高）
aider --model deepseek/deepseek-chat

# 使用通义千问（国内可用）
aider --model openai/qwen-max --openai-api-base https://dashscope.aliyuncs.com/compatible-mode/v1

# 使用本地 Ollama 模型（完全免费离线）
aider --model ollama/qwen2.5-coder:14b
```

### 2. Git 原生集成

Aider 对 Git 的使用是最深入的：

```bash
# 每次 AI 修改后自动创建 commit
# commit message 由 AI 自动生成

# 查看 AI 做了哪些改动
git log --author="aider"

# 不满意？一键回退
git reset --hard HEAD~1
# 然后重新告诉 Aider 你的需求
```

### 3. 代码库地图（Repo Map）

Aider 会生成"代码地图"发给 LLM，帮助模型理解项目全貌：

```
项目结构地图：
├── src/
│   ├── main.py        → 入口文件, 初始化 Flask app
│   ├── models.py      → User, Post 数据模型
│   ├── routes.py      → 路由定义, 调用 services
│   └── services.py    → 业务逻辑层
├── tests/
│   └── test_services.py
└── requirements.txt

函数调用链：
main.py:create_app() → routes.py:register_routes()
                    → models.py:User, Post
```

这个地图让 Aider 远超只看当前文件的工具。

### 4. 多文件同时编辑

```bash
# 一次加载多个文件
aider src/models.py src/services.py src/routes.py

# Aider 可以同时修改这些文件
"给 User 模型加一个 avatar 字段，
 更新 service 层的查询逻辑，
 在路由中返回新字段"
```

### 5. 语音输入

```bash
# 安装语音支持
pip install aider-chat[browser-voice]

# 启动语音模式
aider --voice

# 直接说话给需求，解放双手！
```

## 进阶配置

### `.aider.conf.yml`

```yaml
# 项目级配置
model: claude-3-5-sonnet-20241022

# 自动 commit（默认开启）
auto-commits: true

# 暗色模式
dark-mode: true

# 编辑格式
edit-format: diff

# 代码风格
code-theme: monokai

# 弱模型（用于简单任务，省钱）
weak-model: gpt-4o-mini

# 读取项目 README 获取上下文
read: [README.md, CONVENTIONS.md]
```

### Aider 的约定文件

创建 `CONVENTIONS.md` 告诉 Aider 你的编码习惯：

```markdown
# 编码约定

- Python 使用 f-string 而非 % 或 .format()
- 函数需要有 type hints
- 使用 pathlib 而非 os.path
- 数据库查询使用 SQLAlchemy 2.0 风格
- 所有 public 函数要有 docstring
```

```bash
# 让 Aider 读取约定文件
aider --read CONVENTIONS.md
```

## 与其他工具对比

| 特性 | Aider | Codex CLI | Claude Code |
|------|-------|-----------|-------------|
| 开源 | ✅ Apache 2.0 | ✅ MIT | ❌ |
| 多模型 | ✅ 几乎所有 | ❌ OpenAI | ❌ Anthropic |
| 本地模型 | ✅ Ollama 等 | ❌ | ❌ |
| Git 集成 | ⭐⭐⭐ 自动 commit | ⭐⭐ | ⭐⭐ |
| 多文件编辑 | ✅ | ✅ | ✅ |
| 语音输入 | ✅ | ❌ | ❌ |
| 安装复杂度 | pip install | npm install | npm install |
| 国内可用性 | ✅ 通义千问等 | ⚠️ API 需代理 | ⚠️ API 需代理 |

## 国内使用建议

国内开发者建议使用以下模型：

```bash
# 通义千问（阿里云，国内直连）
export DASHSCOPE_API_KEY="sk-xxx"
aider --model openai/qwen-max \
      --openai-api-base https://dashscope.aliyuncs.com/compatible-mode/v1

# DeepSeek（价格极低）
export DEEPSEEK_API_KEY="sk-xxx"
aider --model deepseek/deepseek-chat

# 本地 Ollama（完全离线免费）
ollama pull qwen2.5-coder:14b
aider --model ollama/qwen2.5-coder:14b
```

## 常见问题

### Q: Aider 安全吗？代码会被上传吗？

代码通过 API 发送给 LLM 提供商。使用本地 Ollama 模型可以完全离线运行，代码不会离开你的机器。

### Q: 如何控制成本？

```bash
# 简单任务用弱模型
aider --model gpt-4o-mini

# 复杂任务用强模型
aider --model claude-3-5-sonnet-20241022

# 用弱模型做 map，强模型做 edit
aider --weak-model deepseek/deepseek-chat
```

### Q: 为什么选择 Aider 而不是 VSCode 插件？

- 终端原生，不需要打开 IDE
- Git 集成更深入（自动 commit）
- 可以结合 Tmux/Screen 创建持久化编程会话
- 支持更多模型（包括本地模型）

## 总结

Aider 的独特优势：

- 🔀 **模型自由**：不绑定任何一家厂商，能省则省，该强则强
- 📝 **Git 原生**：每次改动都是干净的 commit，审查回退极方便
- 🗺 **代码地图**：LLM 能理解整个项目，不只是当前文件
- 🇨🇳 **国内友好**：支持通义千问、DeepSeek，还有本地 Ollama
- 🎙 **语音输入**：说来就来

三者选型建议：
- **需要最强能力 + Anthropic 生态** → Claude Code
- **需要开源 + OpenAI 生态 + Skills** → Codex CLI
- **需要多模型 + 强 Git + 国内可用** → Aider

> 官方仓库：[github.com/paul-gauthier/aider](https://github.com/paul-gauthier/aider)
> 官方文档：[aider.chat](https://aider.chat/)
