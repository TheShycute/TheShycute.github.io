---
layout: post
title: "金属细管内壁缺陷检测系统"
subtitle: "YOLO 深度学习在工业质检中的多模式应用"
date: 2026-05-18 12:00:00
author: "TheShycute"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
  - 项目
  - 深度学习
  - YOLO
  - 工业检测
  - React
---

## 项目背景

金属细管广泛应用于航空航天液压管路、汽车燃油管、医疗器械导管等领域。内壁缺陷（裂纹、凹坑、腐蚀、划痕等）会严重影响管件的承压能力和疲劳寿命，甚至引发安全事故。

传统内窥镜检测依赖人工判断，存在效率低、标准不一的问题。本系统基于 YOLO 深度学习模型，提供了一套覆盖多种场景的自动化检测方案。

## 系统架构

```
┌──────────────────────────────────────────────────┐
│                   前端 (React 18)                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐      │
│  │ 单图检测  │ │ 批量检测  │ │ 实时监测      │      │
│  └──────────┘ └──────────┘ └──────────────┘      │
│  ┌──────────────────────────────────────────┐     │
│  │     检测结果可视化（标注框 + 缺陷分类）       │     │
│  └──────────────────────────────────────────┘     │
├──────────────────────────────────────────────────┤
│                后端服务                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐      │
│  │ 图像处理  │ │ 模型推理  │ │ 报告生成      │      │
│  └──────────┘ └──────────┘ └──────────────┘      │
├──────────────────────────────────────────────────┤
│            YOLO 目标检测模型                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐      │
│  │ 缺陷定位  │ │ 缺陷分类  │ │ 严重程度评估  │      │
│  └──────────┘ └──────────┘ └──────────────┘      │
└──────────────────────────────────────────────────┘
```

## 四种检测模式详解

系统支持灵活的检测模式，适应不同使用场景：

### 📷 模式一：单张图片检测

```
用户上传单张图片 → 预处理 → YOLO 推理 → 标注结果 → 展示
响应时间：< 1 秒
```

适用于快速抽查，操作简单直观。

### 📁 模式二：批量图片检测

```
用户选择多张图片（拖拽或选择文件夹）
    │
    ▼
图片队列管理（进度条显示）
    │
    ├─→ 图片 1 → 检测 → 保存结果
    ├─→ 图片 2 → 检测 → 保存结果
    └─→ 图片 N → 检测 → 保存结果
    │
    ▼
汇总报告（缺陷统计、合格率）
并发处理：最多 4 张同时推理
```

适用于产线抽检、来料检验。

### 🎥 模式三：摄像头实时检测

```
摄像头视频流获取
    │
    ▼
逐帧抽取（每 3 帧取 1 帧）
    │
    ▼
模型推理
    │
    ▼
实时标注叠加 → 屏幕展示
    │
    ▼
缺陷帧自动保存
```

适用于在线检测场景，实时监控。

### 📹 模式四：视频文件检测

```
上传视频文件 → 逐帧抽取 → 批量推理 → 缺陷片段标注 → 报告
支持格式：MP4、AVI、MOV
```

适用于离线分析、回溯检查。

## 前端技术实现

### React 组件设计

```typescript
// 检测模式选择器
const DetectionMode = {
  SINGLE: 'single',
  BATCH: 'batch', 
  CAMERA: 'camera',
  VIDEO: 'video'
};

// 统一检测结果类型
interface DetectionResult {
  mode: DetectionMode;
  defects: DefectItem[];
  stats: {
    total: number;
    defective: number;
    passRate: number;
    avgConfidence: number;
  };
  timestamp: string;
  processingTime: number;
}

interface DefectItem {
  type: 'crack' | 'pit' | 'corrosion' | 'scratch';
  severity: 'mild' | 'moderate' | 'severe';
  location: string;
  confidence: number;
  boundingBox: [number, number, number, number];
}
```

### 状态管理（Zustand）

```typescript
import { create } from 'zustand';

const useDetectionStore = create((set, get) => ({
  mode: 'single',
  images: [],
  results: [],
  isProcessing: false,
  progress: 0,
  
  setMode: (mode) => set({ mode }),
  addImages: (files) => set((state) => ({ 
    images: [...state.images, ...files] 
  })),
  
  processImages: async () => {
    set({ isProcessing: true, progress: 0 });
    const { images } = get();
    const results = [];
    
    for (let i = 0; i < images.length; i++) {
      const result = await detectAPI(images[i]);
      results.push(result);
      set({ progress: ((i + 1) / images.length) * 100 });
    }
    
    set({ results, isProcessing: false });
  },
}));
```

### Webpack 5 构建优化

```javascript
// webpack.config.js
module.exports = {
  mode: 'production',
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader'],
      },
    ],
  },
};
```

## YOLO 模型训练细节

### 数据集构建

| 缺陷类型 | 样本数 | 标注方式 |
|----------|--------|----------|
| 裂纹（Crack） | 1200 | 矩形框 + 多边形 mask |
| 凹坑（Pit） | 1500 | 矩形框 |
| 腐蚀（Corrosion） | 800 | 矩形框 + 语义分割 |
| 划痕（Scratch） | 1000 | 矩形框 |

### 训练策略

- **两阶段训练**：先在公开数据集预训练，再用自制数据集微调
- **数据增强**：随机裁剪、旋转 (±15°)、亮度/对比度调整、高斯噪声
- **损失函数**：CIoU Loss + CrossEntropy Loss
- **学习率调度**：Cosine Annealing with Warmup

### 模型性能

| 指标 | 数值 |
|------|------|
| mAP@0.5 | 94.2% |
| mAP@0.5:0.95 | 78.6% |
| 精确率 | 93.1% |
| 召回率 | 91.8% |
| 推理速度 (GPU) | 15ms/帧 |
| 推理速度 (CPU) | 85ms/帧 |

## 项目收获

1. **多场景适配**：同一模型服务四种检测模式，考验架构设计能力
2. **性能优化**：批量推理的并发控制、摄像头模式的帧率平衡
3. **工程化深度**：Webpack 5 精细化构建配置、TypeScript 严格模式
4. **工业思维**：从精度、速度、稳定性三个维度综合评估系统

> 项目仓库：[Gitee - gwf-bs](https://gitee.com/TheShycute/gwf-bs)（私有）
