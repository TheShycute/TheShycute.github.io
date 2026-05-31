---
layout: post
title: "Python + OpenCV 图像处理快速入门"
subtitle: "从零开始掌握计算机视觉基础"
date: 2026-05-25 10:00:00
author: "TheShycute"
header-img: "img/post-bg-css.jpg"
catalog: true
tags:
  - 教程
  - Python
  - OpenCV
  - 计算机视觉
---

## 为什么学 OpenCV？

OpenCV（Open Source Computer Vision Library）是计算机视觉领域最流行的开源库。无论你是做图像处理、目标检测还是视频分析，OpenCV 都是绕不开的工具。

## 环境搭建

```bash
pip install opencv-python numpy matplotlib
```

验证安装：

```python
import cv2
print(cv2.__version__)  # 例如 4.8.0
```

## 基础操作

### 1. 读取与显示图片

```python
import cv2

img = cv2.imread('photo.jpg')
cv2.imshow('My Image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 2. 灰度化

```python
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```

### 3. 高斯模糊（降噪）

```python
blurred = cv2.GaussianBlur(img, (5, 5), 0)
```

### 4. 边缘检测（Canny）

```python
edges = cv2.Canny(img, 50, 150)
```

### 5. 图像缩放

```python
resized = cv2.resize(img, (300, 300))
```

## 一个完整的图像处理 Pipeline

```python
import cv2

def process_image(path):
    # 读取
    img = cv2.imread(path)
    
    # 灰度化
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    # 高斯模糊
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 边缘检测
    edges = cv2.Canny(blurred, 50, 150)
    
    # 保存结果
    cv2.imwrite('result.jpg', edges)
    return edges
```

## 进阶方向

掌握了基础后，可以往以下方向深入：

| 方向 | 关键技术 |
|------|---------|
| 目标检测 | YOLO、SSD、Faster R-CNN |
| 图像分割 | U-Net、Mask R-CNN |
| 人脸识别 | Haar Cascade、FaceNet |
| OCR | Tesseract、PaddleOCR |
| 视频分析 | 光流法、背景减除 |

## 实践建议

> 看一百遍不如写一遍。

OpenCV 最好的学习方式就是动手实践。试着做一个车牌识别、人脸检测或是简单的滤镜效果，你很快就能掌握核心概念。

> 参考：[GitHub - License Plate Recognition](https://github.com/TheShycute/License_Plate_Recognition)
