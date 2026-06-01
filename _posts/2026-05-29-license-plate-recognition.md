---
layout: post
title: "YOLO11 + CRNN 全栈车牌识别系统 v2.0 正式发布"
subtitle: "玻璃拟态UI · 三级权限 · 批量/视频/摄像头检测 · 开源发布"
date: 2026-06-02 08:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
  - 教程
  - 计算机视觉
  - Python
  - YOLO
  - Flask
  - 开源
---

## 项目概述

车牌识别系统迎来 **v2.0 重磅升级**——从命令行工具进化为全栈 Web 应用，正式以 **AGPL-3.0** 许可证在 GitHub 开源。

> 📂 GitHub 仓库：[TheShycute/License_Plate_Recognition](https://github.com/TheShycute/License_Plate_Recognition)

### 技术栈
- **AI 检测**: YOLO11 (Ultralytics)
- **AI 识别**: CRNN (PyTorch)
- **后端**: Flask 3.x + SQLite + JWT 认证
- **前端**: 原生 HTML/CSS/JS，白色玻璃拟态设计
- **推理**: CUDA 12.6 + PyTorch 2.9

---

## 系统截图

### 登录页面
白色玻璃拟态设计，粒子网络动画背景，三级权限登录。

![登录页](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/01_login.png)

### 主面板（单张检测）
上传区 → 选择图片 → 预览 → AI 识别，流畅三步体验。

![主面板](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/02_dashboard.png)

### 检测结果
两列布局：左侧图片区（支持结果图/原图切换），右侧车牌详情卡片。

![检测结果](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/03_result.png)

### 原图切换
点击「原始图片」标签可随时切换查看未标注的原图。

![原图切换](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/04_origin.png)

### 批量检测
一次上传最多 20 张图片，逐张可展开查看结果。

![批量检测](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/05_batch.png)

### 视频检测
上传 MP4/AVI/MOV 等格式视频，逐帧检测生成标注结果视频。

![视频检测](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/06_video.png)

### 摄像头实时检测
浏览器摄像头采集，支持手动拍照或每秒自动检测。

![摄像头检测](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/07_camera.png)

### 历史记录
自动保存所有检测记录，支持搜索、日期筛选、分页浏览。

![历史记录](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/08_history.png)

### 检测详情弹窗
弹窗展示完整车牌信息：车牌号、颜色色块、置信度进度条、结果图片。

![检测详情](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/09_detail.png)

### 用户管理
管理员可创建/管理用户，支持三级权限分配。

![用户管理](https://raw.githubusercontent.com/TheShycute/License_Plate_Recognition/master/screenshots/10_users.png)

---

## 功能特性

### 检测模式
| 模式 | 说明 |
|------|------|
| 📷 单张检测 | 上传图片，实时返回车牌号、颜色、置信度 |
| 📚 批量检测 | 一次处理最多 20 张图片 |
| 🎬 视频检测 | 上传视频文件，逐帧检测生成标注视频 |
| 📹 摄像头 | 浏览器摄像头采集，支持手动/自动模式 |

### 系统功能
| 功能 | 说明 |
|------|------|
| 🔐 三级权限 | 管理员 / 操作员 / 查看者 |
| 📋 历史记录 | 自动保存，支持搜索、筛选、批量删除 |
| 👥 用户管理 | 管理员可增删用户 |
| 🎨 现代 UI | 玻璃拟态设计，粒子动画背景 |

---

## 部署指南

### 第一步：克隆项目
```bash
git clone https://github.com/TheShycute/License_Plate_Recognition.git
cd License_Plate_Recognition
```

### 第二步：创建虚拟环境
```bash
# 使用 conda（推荐）
conda create -n yolov11 python=3.10 -y
conda activate yolov11
```

### 第三步：安装 PyTorch（CUDA 版本）
```bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu126
```

### 第四步：安装项目依赖
```bash
pip install -r requirements_web.txt
```

### 第五步：启动服务
```bash
python app.py
```

浏览器访问 **http://localhost:5001**

### 默认账号
| 用户名 | 密码 | 角色 |
|--------|------|------|
| admin | admin123 | 管理员（全部功能） |
| operator | op123 | 操作员（检测+历史） |
| viewer | view123 | 查看者（仅历史） |

---

## 项目结构
```
├── app.py                         # Flask 全栈入口
├── detect_rec_plate.py            # 检测核心 + CLI
├── plate_recognition/             # 识别模块
├── fonts/                         # 中文字体渲染
├── weights/                       # 模型权重
│   ├── yolo26s-plate-detect.pt    # YOLO11 (10.58M 参数)
│   └── plate_rec_color.pth        # CRNN (0.18M 参数)
├── templates/                     # HTML 模板
├── static/                        # CSS/JS 前端资源
├── screenshots/                   # 系统截图
└── docs/blog.md                   # 开发博客
```

---

## 性能数据

测试环境：**RTX 3060 Laptop GPU (6GB) + CUDA 12.6**

| 指标 | 数值 |
|------|------|
| 单图检测耗时 | ~85ms |
| 批量检测（20张） | ~1.7s |
| 检测模型大小 | 61MB |
| 识别模型大小 | 0.7MB |
| 内存占用 | ~1.2GB |

---

## 权限设计

| 角色 | 单张 | 批量 | 视频 | 摄像头 | 历史 | 用户管理 |
|------|:--:|:--:|:--:|:--:|:--:|:--:|
| 管理员 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 操作员 | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| 查看者 | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |

---

## 开源信息
- **仓库**: [TheShycute/License_Plate_Recognition](https://github.com/TheShycute/License_Plate_Recognition)
- **许可证**: AGPL-3.0
- **语言**: Python / JavaScript / CSS / HTML

---

## 致谢
- [Ultralytics YOLO](https://github.com/ultralytics/ultralytics)
- [PyTorch](https://pytorch.org)
- [Flask](https://flask.palletsprojects.com)
