---
layout: post
title: "从零搭建 GitHub Pages 个人博客"
subtitle: "基于 Hux Blog 主题的免费博客方案"
date: 2026-05-31 12:00:00
author: "TheShycute"
header-img: "img/home-bg-geek.jpg"
catalog: true
tags:
  - 教程
  - GitHub Pages
  - Jekyll
  - 博客
---

## 为什么选择 GitHub Pages

GitHub Pages 是 GitHub 提供的免费静态网站托管服务，优势明显：

- 🆓 **完全免费** — 无限流量，无广告
- 🚀 **自动部署** — 推送代码即自动构建发布
- 🎨 **主题丰富** — 社区有大量开源 Jekyll 主题
- 🔒 **HTTPS 支持** — 自动配置 SSL 证书

## 搭建步骤

### 1. 创建仓库

创建名为 `<username>.github.io` 的仓库（我的就是 `TheShycute.github.io`）。

### 2. 选择主题

我选择了 [Hux Blog](https://github.com/Huxpro/huxpro.github.io) 主题，这是一个非常经典的 Jekyll 博客主题，特点：

- 简约现代的设计风格
- 支持标签分类
- 内置搜索功能
- PWA 支持
- 响应式布局

### 3. 个性化配置

修改 `_config.yml` 文件：

```yaml
title: TheShycute Blog
url: "https://theshycute.github.io"
github_username: TheShycute
```

### 4. 撰写文章

在 `_posts/` 目录下创建 Markdown 文件，命名格式为 `YYYY-MM-DD-title.md`。

文章头部需要包含 YAML Front Matter：

```yaml
---
layout: post
title: "你的文章标题"
date: 2026-05-31 12:00:00
tags:
  - 标签1
  - 标签2
---
```

### 5. 推送发布

```bash
git add -A
git commit -m "新文章"
git push origin master
```

几分钟后，博客就会自动更新！

## 写在最后

搭建博客只是开始，坚持写作才是关键。用博客记录学习历程，既是自我沉淀，也是一种分享。

> 🔗 博客地址：[theshycute.github.io](https://theshycute.github.io)
> 📂 源代码：[GitHub 仓库](https://github.com/TheShycute/TheShycute.github.io)
