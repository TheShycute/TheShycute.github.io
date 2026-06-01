---
layout: post
title: "基于YOLO26的金属细管内壁缺陷检测系统 - 全栈实战"
subtitle: "YOLO26 + FastAPI + React 18 + TypeScript 工业缺陷检测全栈方案"
date: 2026-05-30 12:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
  - 项目
  - 深度学习
  - YOLO26
  - 工业检测
  - FastAPI
  - React
  - TypeScript
---

## 项目概述

金属细管内壁缺陷检测系统是一个完整的工业质检全栈解决方案，前后端代码逾 **18000 行**，支持单张图片、批量图片、摄像头实时、视频文件离线四种检测模式。

> 项目仓库：[Gitee - gwfBS/f5](https://gitee.com/TheShycute/gwfBS)

## 技术栈

| 层级 | 技术选型 |
|------|----------|
| 前端框架 | React 18 + TypeScript + Webpack 5 |
| 样式方案 | Tailwind CSS + CSS 自定义属性 |
| UI 组件 | Lucide React + Framer Motion 动画 |
| 图表库 | Recharts |
| 后端框架 | Python FastAPI |
| AI 推理 | YOLO26 (ultralytics) + PyTorch + TensorFlow + ONNX |
| 图像处理 | OpenCV |
| 数据库 | MySQL（支持 Supabase 云数据库切换） |
| 认证授权 | JWT（admin / operator / viewer 三级角色） |
| 测试框架 | pytest + httpx |
| 部署方案 | Nginx 反向代理 + Uvicorn + Docker |

## 系统架构

```
+-------------------------------------------------------------+
|                     前端 (React 18 + TypeScript)              |
|  +----------+ +----------+ +----------+ +----------+       |
|  | 单图检测  | | 批量检测  | | 摄像头检测| | 视频检测  |       |
|  +----------+ +----------+ +----------+ +----------+       |
|  +------------------------------------------------------+   |
|  |  检测历史管理 | 数据统计分析 | 用户管理 | 系统设置    |   |
|  +------------------------------------------------------+   |
+-------------------------------------------------------------+
|                    Nginx 反向代理 (:8080)                     |
+-------------------------------------------------------------+
|                   后端 API (FastAPI :8002)                    |
|  +------------+ +------------+ +----------------------+     |
|  | 认证服务    | | 检测服务    | | AI模型管理服务        |     |
|  | JWT Token  | | CRUD操作   | | YOLO/PyTorch/TF/ONNX |     |
|  +------------+ +------------+ +----------------------+     |
|  +----------------------------------------------------+     |
|  |  摄像头流服务 (RTSP/MJPEG/USB) | 视频检测服务       |     |
|  +----------------------------------------------------+     |
+-------------------------------------------------------------+
|                    MySQL / Supabase                          |
|  +----------+ +----------+ +----------+ +----------+       |
|  | users    | |detection | | models   | | camera   |       |
|  |          | |_records  | |          | | _logs    |       |
|  +----------+ +----------+ +----------+ +----------+       |
+-------------------------------------------------------------+
```

## 四种检测模式

### 单张图片检测

- 支持拖拽上传、粘贴图片
- Canvas 标注叠加，缺陷框 + 置信度实时显示
- 响应时间 < 1 秒

### 批量图片检测

- 并发处理，进度条实时追踪
- 批次管理（创建/暂停/恢复/查看）
- 汇总报告（缺陷统计、合格率、耗时分析）

### 摄像头实时检测

- 支持 RTSP / MJPEG / USB 多协议接入
- OpenCV 流管理，自动重连机制
- 实时推理标注 + 缺陷弹窗告警
- 检测日志选择性持久化保存

### 视频文件检测

- 上传 MP4/AVI/MOV 格式
- OpenCV 抽帧 + YOLO 批量检测
- 边播放边标注，支持跳帧、暂停、断点续传

## 后端核心服务设计

### 多框架模型管理

`model_service.py` 实现了统一的模型加载接口，支持四种格式动态切换：

- `*.pt` -> YOLO (ultralytics)
- `*.pt/*.pth` -> PyTorch (torch.load)
- `*.pb/*.h5` -> TensorFlow (saved_model/keras)
- `*.onnx` -> ONNX Runtime

内置 12 类缺陷类别：凸起、焊缝、裂纹、腐蚀、点蚀、划痕、凹痕、磨损、锈蚀、孔洞、变形、其他。置信度阈值、IOU 阈值可动态调节。

### 双数据库后端

`detection_service.py` 实现了 MySQL 和 Supabase 云数据库的统一数据访问抽象层，上层业务代码无感知切换。

### 摄像头流管理

`camera_service.py` 自动检测流类型（RTSP / MJPEG / USB），支持多流并发管理与断线自动重连。

## 前端核心组件

| 页面 | 代码行数 | 核心功能 |
|------|----------|----------|
| CameraDetectionPage | 2896 行 | 摄像头检测，RTSP流管理，实时标注，弹窗告警 |
| HistoryPage | 2092 行 | 检测历史，时间筛选，CSV/JSON导出 |
| BatchDetectionPage | 1610 行 | 批量检测，进度追踪，批次管理 |
| SystemSettingsPage | 1207 行 | 系统配置，模型切换，类别管理 |
| SingleDetectionPage | 1060 行 | 单图检测，拖拽上传，Canvas标注 |
| StatisticsPage | 904 行 | 每日统计，缺陷类型分布，趋势分析 |

## 数据库设计

系统设计了 8 张核心数据表：

- **users** - 用户表（admin / operator / viewer 三级角色）
- **detection_records** - 检测记录（single / batch / camera / video）
- **detection_defects** - 缺陷明细
- **camera_logs** - 摄像头实时检测日志
- **models** - AI 模型版本管理
- **system_settings** - 系统配置（阈值、模型路径等）
- **notifications** - 系统通知
- **batches** - 批量检测任务

## 认证与权限

基于 JWT 的三级角色权限控制，50+ 个 RESTful API 端点：

| 角色 | 权限范围 |
|------|----------|
| admin | 全部权限，可管理用户和系统设置 |
| operator | 可执行所有检测操作 |
| viewer | 仅可查看历史记录和统计数据 |

## 自动化测试

基于 pytest + httpx 的完整测试套件，覆盖 8 个模块：

- 认证模块（登录/Token验证）
- 单张检测模块、批量检测模块
- 摄像头检测模块
- 统计模块、用户管理模块
- 通知模块、系统设置模块

## 模型训练

### 数据集

| 缺陷类型 | 样本数 |
|----------|--------|
| 凸起 | 1200 |
| 焊缝 | 1500 |
| 裂纹 | 800 |
| 腐蚀 | 1000 |
| 点蚀 | 600 |
| 划痕 | 900 |

### 训练策略

- 迁移学习 + 微调
- 数据增强：随机裁剪、旋转、亮度/对比度调整、高斯噪声
- 损失函数：CIoU Loss + CrossEntropy Loss
- 学习率：Cosine Annealing with Warmup

### 模型性能

| 指标 | 数值 |
|------|------|
| mAP@0.5 | 94.2% |
| 精确率 | 93.1% |
| 召回率 | 91.8% |
| GPU推理 | 15ms/帧 |
| CPU推理 | 85ms/帧 |

## 部署方案

```bash
# 一键启动（Linux）
./start-all.sh

# 分别启动
./start-backend.sh    # FastAPI :8002
./start-frontend.sh   # Webpack Dev Server :3015

# 代理（统一入口 :8080）
python3 proxy_server.py

# 生产部署
pnpm run build
python -m uvicorn api.main:app --host 0.0.0.0 --port 8002
cp nginx.conf /etc/nginx/sites-available/
```

## 项目收获

1. **全栈架构能力**：独立设计并实现了从前端界面、后端API、AI模型管理到数据库设计的完整系统
2. **多框架适配**：支持 YOLO / PyTorch / TensorFlow / ONNX 四种模型格式，锻炼了框架兼容性设计能力
3. **工程化实践**：TypeScript 严格模式、Webpack 精细配置、pytest 自动化测试、Nginx 生产部署
4. **性能优化**：从批量推理并发控制到摄像头模式跳帧策略，在精度和速度间找到平衡
5. **工业思维**：三级权限管理、双数据库后端、系统设置热更新，面向真实工业场景设计

> 项目仓库：[Gitee - gwfBS/f5](https://gitee.com/TheShycute/gwfBS)
