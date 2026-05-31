---
layout: post
title: "车牌识别系统 License Plate Recognition"
subtitle: "计算机视觉在智能交通中的应用"
date: 2026-04-01 12:00:00
author: "TheShycute"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
  - 项目
  - 计算机视觉
  - Python
  - OpenCV
  - GitHub
---

## 项目概述

车牌识别（License Plate Recognition, LPR）是智能交通系统的核心技术之一，广泛应用于停车场管理、交通违章抓拍、高速 ETC 等场景。

本项目实现了一套完整的车牌检测与识别流程。

## 实现流程

```
图像输入 → 车牌定位 → 字符分割 → 字符识别 → 结果输出
```

### 1. 车牌定位

使用图像处理技术定位车牌区域：
- 灰度化与高斯模糊
- 边缘检测（Canny / Sobel）
- 形态学操作（膨胀、腐蚀）
- 轮廓查找与筛选

### 2. 字符分割

将车牌区域中的字符逐一分割：
- 二值化处理
- 投影法分割
- 连通域分析

### 3. 字符识别

- 模板匹配
- 或基于深度学习的 OCR（如 CRNN）

## 技术栈

- **语言**：Python
- **图像处理**：OpenCV
- **数值计算**：NumPy
- **可选深度学习**：TensorFlow / PyTorch

## 项目价值

通过这个项目，我深入理解了计算机视觉中图像预处理、特征提取和模式识别的完整流程，为后续深度学习项目打下了坚实基础。

> 项目仓库：[GitHub - License_Plate_Recognition](https://github.com/TheShycute/License_Plate_Recognition)
