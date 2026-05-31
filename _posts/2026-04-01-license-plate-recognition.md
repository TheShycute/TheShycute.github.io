---
layout: post
title: "车牌识别系统 License Plate Recognition"
subtitle: "从图像处理到字符识别的完整实现"
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

车牌识别（LPR, License Plate Recognition）是智能交通系统（ITS）的核心组成，应用覆盖停车场管理、高速 ETC、违章抓拍、安防监控等。

本项目从零实现了一套完整的车牌检测与识别流水线。

## 整体 Pipeline

```
原始图像
    │
    ▼
① 预处理（灰度化、去噪、增强）
    │
    ▼
② 车牌定位（边缘检测 + 形态学 + 轮廓筛选）
    │
    ▼
③ 透视校正（倾斜车牌矫正）
    │
    ▼
④ 字符分割（投影法 + 连通域分析）
    │
    ▼
⑤ 字符识别（模板匹配 / CNN）
    │
    ▼
输出结果
```

## 详细实现

### ① 图像预处理

```python
import cv2
import numpy as np

def preprocess(image_path):
    # 读取图像
    img = cv2.imread(image_path)
    
    # 灰度化
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    # 高斯模糊去噪
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 直方图均衡化增强对比度
    equalized = cv2.equalizeHist(blurred)
    
    return img, equalized
```

### ② 车牌定位

```python
def locate_plate(processed_img):
    # 边缘检测
    edges = cv2.Canny(processed_img, 100, 200)
    
    # 形态学操作（闭运算连接边缘）
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (17, 5))
    closed = cv2.morphologyEx(edges, cv2.MORPH_CLOSE, kernel)
    
    # 查找轮廓
    contours, _ = cv2.findContours(
        closed, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE
    )
    
    # 筛选候选区域（基于车牌宽高比和面积）
    candidates = []
    for cnt in contours:
        x, y, w, h = cv2.boundingRect(cnt)
        aspect_ratio = w / h
        area = w * h
        
        # 中国蓝牌：宽高比约 3.14:1
        if 2.0 < aspect_ratio < 5.5 and area > 2000:
            candidates.append((x, y, w, h, area))
    
    # 选面积最大的作为车牌区域
    if candidates:
        candidates.sort(key=lambda x: x[4], reverse=True)
        return candidates[0][:4]  # (x, y, w, h)
    return None
```

### ③ 透视校正

```python
def correct_perspective(img, plate_rect):
    x, y, w, h = plate_rect
    plate_roi = img[y:y+h, x:x+w]
    
    # 转为灰度进行角点检测
    gray_roi = cv2.cvtColor(plate_roi, cv2.COLOR_BGR2GRAY)
    
    # 二值化
    _, binary = cv2.threshold(gray_roi, 0, 255, 
                               cv2.THRESH_BINARY + cv2.THRESH_OTSU)
    
    # 查找车牌四个角点（简化为基于轮廓的最小外接矩形）
    contours, _ = cv2.findContours(binary, cv2.RETR_EXTERNAL, 
                                    cv2.CHAIN_APPROX_SIMPLE)
    
    if contours:
        largest = max(contours, key=cv2.contourArea)
        rect = cv2.minAreaRect(largest)
        box = cv2.boxPoints(rect)
        box = np.int0(box)
        
        # 透视变换
        width, height = 440, 140  # 标准车牌尺寸比
        dst_points = np.array([[0, 0], [width, 0], 
                               [width, height], [0, height]], 
                             dtype=np.float32)
        M = cv2.getPerspectiveTransform(box.astype(np.float32), dst_points)
        corrected = cv2.warpPerspective(plate_roi, M, (width, height))
        
        return corrected
    
    return plate_roi  # 无法校正则返回原图
```

### ④ 字符分割

```python
def segment_characters(plate_img):
    gray = cv2.cvtColor(plate_img, cv2.COLOR_BGR2GRAY)
    
    # 二值化（黑底白字 → 白底黑字）
    _, binary = cv2.threshold(gray, 0, 255, 
                               cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
    
    # 垂直投影
    hist = np.sum(binary, axis=0)
    
    # 找到字符区域
    chars = []
    in_char = False
    start = 0
    
    for i, val in enumerate(hist):
        if val > 0 and not in_char:
            in_char = True
            start = i
        elif val == 0 and in_char:
            in_char = False
            width = i - start
            if width > 3:  # 过滤噪点
                chars.append(binary[:, start:i])
    
    return chars
```

### ⑤ 字符识别

```python
def recognize_char(char_img, templates):
    """模板匹配识别单个字符"""
    best_match = None
    best_score = 0
    
    # 统一尺寸
    char_resized = cv2.resize(char_img, (20, 30))
    
    for label, template in templates.items():
        template_resized = cv2.resize(template, (20, 30))
        
        # 计算相似度
        score = cv2.matchTemplate(char_resized, template_resized, 
                                   cv2.TM_CCOEFF_NORMED)[0][0]
        
        if score > best_score:
            best_score = score
            best_match = label
    
    return best_match if best_score > 0.5 else '?'

def recognize_plate(chars, templates):
    """识别整个车牌"""
    result = ''
    for i, char_img in enumerate(chars):
        ch = recognize_char(char_img, templates)
        result += ch
    
    return result
```

## 进阶方案：基于深度学习的 OCR

模板匹配在光照、角度变化大的场景下表现一般。进阶方案可采用：

- **CRNN**（卷积循环神经网络）：CNN 提取特征 + RNN 序列建模 + CTC 损失
- **PaddleOCR**：百度开源的中文 OCR 工具包，开箱即用
- **EasyOCR**：支持 80+ 语言的轻量 OCR

```python
# 使用 EasyOCR 的简化示例
import easyocr

reader = easyocr.Reader(['ch_sim', 'en'])
results = reader.readtext('plate.jpg')
for bbox, text, confidence in results:
    print(f"识别结果: {text}, 置信度: {confidence:.2f}")
```

## 项目总结

通过这个项目，我掌握了：

- OpenCV 图像处理的完整流程
- 边缘检测、形态学操作、轮廓分析等核心技巧
- 从传统模板匹配到深度学习 OCR 的技术演进路线
- 计算机视觉项目的工程化实践

车牌识别看似简单，但要在真实场景中达到工业级精度（>99%），需要考虑光照、角度、遮挡、污损等各种情况。这也是 CV 项目的魅力所在——理论和实践之间的鸿沟需要一点一点填平。

> 项目仓库：[GitHub - License_Plate_Recognition](https://github.com/TheShycute/License_Plate_Recognition)
