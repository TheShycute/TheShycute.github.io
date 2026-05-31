---
layout: post
title: "GitHub Pages 个人博客搭建全攻略"
subtitle: "基于 Hux Blog 主题的免费博客方案"
date: 2026-05-20 12:00:00
author: "天使皮卡丘丨"
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
date: 2026-05-20 12:00:00
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


## 额外配置：SEO 与订阅

### 站点地图 sitemap.xml

创建 `sitemap.xml` 帮助搜索引擎收录：

```xml
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
{{ '% for post in site.posts %' }}
  <url>
    <loc>{{ '{{ site.url }}{{ post.url }}' }}</loc>
    <lastmod>{{ '{{ post.date | date_to_xmlschema }}' }}</lastmod>
    <priority>0.8</priority>
  </url>
{{ '% endfor %' }}
</urlset>
```

### robots.txt

```
User-agent: *
Allow: /
Sitemap: https://你的用户名.github.io/sitemap.xml
```

### RSS 订阅

博客内置 RSS 功能，读者访问 `/feed.xml` 即可获取订阅源。用任意 RSS 阅读器添加这个链接，就能在新文章发布时自动收到通知。

> 📡 本站 RSS：[https://theshycute.github.io/feed.xml](https://theshycute.github.io/feed.xml)

## 写在最后

搭建博客只是开始，坚持写作才是关键。用博客记录学习历程，既是自我沉淀，也是一种分享。

> 🔗 博客地址：[theshycute.github.io](https://theshycute.github.io)
> 📂 源代码：[GitHub 仓库](https://github.com/TheShycute/TheShycute.github.io)
