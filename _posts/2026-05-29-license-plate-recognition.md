---
layout: post
title: "基于 OpenCV 的车牌识别系统设计与实现"
subtitle: "手把手教你从零搭建一个完整车牌识别系统"
date: 2026-05-29 12:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
  - 教程
  - 计算机视觉
  - Python
  - OpenCV
  - GitHub
---

## 项目概述

车牌识别（License Plate Recognition，LPR）是智能交通系统的核心技术之一。本文将从零开始，带你用 Python + OpenCV 实现一个完整的车牌识别系统。

**学习目标：**
- 掌握 OpenCV 图像处理的核心 API
- 理解车牌定位、字符分割、字符识别的完整流程
- 能够独立复现并改进这个系统

> 📂 完整代码：[GitHub - License_Plate_Recognition](https://github.com/TheShycute/License_Plate_Recognition)

## 环境准备

### 安装依赖

```bash
pip install opencv-python numpy matplotlib
```

验证安装：

```python
import cv2
import numpy as np
print(f"OpenCV 版本: {cv2.__version__}")
print(f"NumPy 版本: {np.__version__}")
```

### 项目结构

```
license_plate_recognition/
├── images/              # 测试图片
├── templates/           # 字符模板
├── src/
│   ├── preprocess.py    # 图像预处理
│   ├── locate.py        # 车牌定位
│   ├── segment.py       # 字符分割
│   ├── recognize.py     # 字符识别
│   └── pipeline.py      # 完整流水线
├── main.py              # 入口文件
└── README.md
```

---

## 第一步：图像预处理

预处理的目标是**增强车牌区域特征，抑制噪声**，为后续定位创造条件。

```python
import cv2
import numpy as np

def preprocess(image_path):
    """
    图像预处理流水线
    输入：原始图像路径
    输出：原始 BGR 图、预处理后的灰度图
    """
    # 1. 读取图像
    img = cv2.imread(image_path)
    if img is None:
        raise FileNotFoundError(f"无法读取图像: {image_path}")
    
    # 2. 灰度化——将三通道转为单通道
    #    公式：Gray = 0.299*R + 0.587*G + 0.114*B
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    # 3. 高斯模糊——平滑图像，去除高频噪声
    #    核大小 (5,5)：太小去噪不够，太大会模糊边缘
    #    sigmaX=0：自动计算标准差
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 4. 直方图均衡化——增强对比度
    #    让像素值分布更均匀，车牌区域更突出
    equalized = cv2.equalizeHist(blurred)
    
    return img, equalized
```

### 各步骤效果对比

| 步骤 | 作用 | 为什么需要 |
|------|------|-----------|
| **灰度化** | BGR → 单通道 | 降低计算量，大多数 CV 算法只接受灰度图 |
| **高斯模糊** | 平滑降噪 | 去除摄像头/传感器产生的高频噪点 |
| **直方图均衡化** | 拉伸对比度 | 改善光照不均，让车牌边缘更明显 |

---

## 第二步：车牌定位

这是整个系统最关键的步骤——**在一张照片里找到车牌的位置**。

### 核心思路

中国蓝牌有明确的视觉特征：矩形、蓝底白字（灰度图中表现为特定的明暗对比）。利用这些特征来定位。

```python
def locate_plate(processed_img):
    """
    车牌定位
    输入：预处理后的灰度图
    输出：车牌区域坐标 (x, y, w, h) 或 None
    """
    # === 1. 边缘检测 ===
    # Canny 算法：双阈值 + 滞后跟踪
    # threshold1=100：低阈值（低于此值的边缘被丢弃）
    # threshold2=200：高阈值（高于此值的边缘被保留）
    edges = cv2.Canny(processed_img, 100, 200)
    
    # === 2. 形态学操作 ===
    # 车牌字符之间有间隙，通过闭运算（先膨胀后腐蚀）把它们连成一片
    # 核大小 (17, 5)：水平方向宽（车牌长）、垂直方向窄（车牌扁）
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (17, 5))
    closed = cv2.morphologyEx(edges, cv2.MORPH_CLOSE, kernel)
    
    # === 3. 查找轮廓 ===
    contours, _ = cv2.findContours(
        closed, 
        cv2.RETR_EXTERNAL,      # 只取最外层轮廓
        cv2.CHAIN_APPROX_SIMPLE  # 压缩冗余点
    )
    
    # === 4. 筛选候选区域 ===
    candidates = []
    for cnt in contours:
        x, y, w, h = cv2.boundingRect(cnt)
        aspect_ratio = w / h   # 宽高比
        area = w * h           # 面积
        
        # 中国蓝牌宽高比约 3.14:1（440mm × 140mm）
        # 考虑到拍摄角度，给一个宽松范围
        if 2.0 < aspect_ratio < 5.5 and area > 2000:
            candidates.append((x, y, w, h, area))
    
    if not candidates:
        return None
    
    # 选面积最大的作为车牌区域
    candidates.sort(key=lambda x: x[4], reverse=True)
    return candidates[0][:4]  # 返回 (x, y, w, h)
```

### 算法详解

**Canny 边缘检测的三步过程：**

```
① 计算梯度幅值和方向 (Sobel 算子)
    ↓
② 非极大值抑制（保留梯度方向上的最大值，"细化"边缘）
    ↓
③ 双阈值滞后跟踪
    ├─ > 高阈值 → 强边缘（保留）
    ├─ 高低之间 → 弱边缘（仅当连接强边缘时保留）
    └─ < 低阈值 → 丢弃
```

**形态学闭运算：**

```
原图边缘（字符间隙）
  ██   ██   ██   ██   ██
  
膨胀（扩大白色区域）
  ████████████████████████
  
腐蚀（缩小白色区域，但连通区域保持）
  ██████████████████████
  
→ 一个完整的矩形区域！
```

### 定位结果可视化

```python
def visualize_locate(img, plate_rect):
    """在原始图像上绘制定位结果"""
    if plate_rect is None:
        print("未检测到车牌！")
        return img
    
    x, y, w, h = plate_rect
    result = img.copy()
    
    # 绘制绿色矩形框
    cv2.rectangle(result, (x, y), (x + w, y + h), (0, 255, 0), 2)
    
    # 添加标签
    cv2.putText(result, 'Plate', (x, y - 10),
                cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2)
    
    return result
```

---

## 第三步：透视校正

拍摄角度倾斜会导致字符变形，校正后能显著提升识别率。

```python
def correct_perspective(img, plate_rect):
    """
    透视校正：将倾斜的车牌"拉正"
    输入：原始 BGR 图像、车牌区域 (x, y, w, h)
    输出：校正后的车牌图像
    """
    x, y, w, h = plate_rect
    
    # 裁取车牌区域（稍微扩大边界避免裁切到边缘）
    margin = 3
    y1 = max(0, y - margin)
    y2 = min(img.shape[0], y + h + margin)
    x1 = max(0, x - margin)
    x2 = min(img.shape[1], x + w + margin)
    plate_roi = img[y1:y2, x1:x2]
    
    # 转为灰度并二值化
    gray_roi = cv2.cvtColor(plate_roi, cv2.COLOR_BGR2GRAY)
    _, binary = cv2.threshold(gray_roi, 0, 255,
                               cv2.THRESH_BINARY + cv2.THRESH_OTSU)
    
    # 查找车牌轮廓，获取最小外接矩形（含旋转角度）
    contours, _ = cv2.findContours(binary, cv2.RETR_EXTERNAL,
                                    cv2.CHAIN_APPROX_SIMPLE)
    
    if not contours:
        return plate_roi  # 无法校正，返回原图
    
    largest = max(contours, key=cv2.contourArea)
    rect = cv2.minAreaRect(largest)
    box = cv2.boxPoints(rect)
    box = np.int32(box)
    
    # 透视变换
    width, height = 440, 140  # 标准车牌宽高
    dst_points = np.array([
        [0, 0], [width - 1, 0],
        [width - 1, height - 1], [0, height - 1]
    ], dtype=np.float32)
    
    src_points = box.astype(np.float32)
    M = cv2.getPerspectiveTransform(src_points, dst_points)
    corrected = cv2.warpPerspective(plate_roi, M, (width, height))
    
    return corrected
```

### 透视变换原理

```
倾斜视角                          正视视角
  ┌──────┐                        ┌──────────┐
  │ ╱╲  │                        │  京A12345 │
  │ ╱  ╲ │      ────→            │           │
  │╱    ╲│      透视变换          └──────────┘
  └──────┘
  
变换矩阵 M 由四个角点的对应关系解出：
  dst = M · src
```

---

## 第四步：字符分割

将车牌上的字符一个一个切出来。

```python
def segment_characters(plate_img):
    """
    字符分割：将车牌图像的7个字符逐一分离
    输入：校正后的车牌图像（BGR）
    输出：字符图像列表
    """
    gray = cv2.cvtColor(plate_img, cv2.COLOR_BGR2GRAY)
    
    # 二值化（白底黑字 → 黑底白字）
    # 用 OTSU 自动阈值，避免手动调参
    _, binary = cv2.threshold(gray, 0, 255,
                               cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
    
    # 垂直投影——统计每列白色像素数量
    hist = np.sum(binary // 255, axis=0)
    
    # 基于投影分割字符
    chars = []
    in_char = False
    start = 0
    
    for i, val in enumerate(hist):
        if val > 2 and not in_char:
            in_char = True
            start = i
        elif val <= 2 and in_char:
            in_char = False
            width = i - start
            if width > 3:  # 过滤噪点（宽度太小的不是字符）
                char_img = binary[:, start:i]
                chars.append(char_img)
    
    return chars
```

### 垂直投影示意图

```
车牌图像：
  ██ ██ ███ ████ ██ ███ ██ ██
  
垂直投影（每列白色像素数）：
  ██     ██     ██     ██
  ██ ██  ██  ██ ██  ██ ██
  ██ ██  ██  ██ ██  ██ ██
  ─────────────────────────
  京   A   1   2   3   4   5
  
波谷（白像素少）→ 字符间隙
波峰（白像素多）→ 字符本身
```

---

## 第五步：字符识别

### 方案一：模板匹配

简单直观，适合标准字体。

```python
import os

def load_templates(templates_dir):
    """
    加载字符模板库
    目录结构：templates/0.png, templates/1.png, ... templates/Z.png
    """
    templates = {}
    for filename in os.listdir(templates_dir):
        if filename.endswith(('.png', '.jpg')):
            label = filename.split('.')[0]  # 文件名即标签
            path = os.path.join(templates_dir, filename)
            template = cv2.imread(path, cv2.IMREAD_GRAYSCALE)
            if template is not None:
                templates[label] = template
    return templates

def recognize_char(char_img, templates):
    """
    模板匹配识别单个字符
    输入：字符二值图、模板字典 {label: image}
    输出：识别结果字符、置信度
    """
    best_match = '?'
    best_score = 0
    
    # 统一尺寸
    char_resized = cv2.resize(char_img, (20, 30))
    
    for label, template in templates.items():
        template_resized = cv2.resize(template, (20, 30))
        
        # 模板匹配：在模板图中找与字符图最相似的区域
        # TM_CCOEFF_NORMED：归一化相关系数，值越大越相似
        result = cv2.matchTemplate(char_resized, template_resized,
                                    cv2.TM_CCOEFF_NORMED)
        score = result[0][0]
        
        if score > best_score:
            best_score = score
            best_match = label
    
    return best_match, best_score

def recognize_plate(chars, templates):
    """
    识别整个车牌
    输入：字符图像列表、模板字典
    输出：车牌号字符串、每个字符的置信度
    """
    result = ''
    confidences = []
    
    for char_img in chars:
        ch, conf = recognize_char(char_img, templates)
        result += ch
        confidences.append(conf)
    
    return result, confidences
```

### 方案二：EasyOCR（深度学习方案）

模板匹配在复杂环境下精度有限，深度学习方案更鲁棒：

```python
import easyocr

# 初始化（只需一次）
reader = easyocr.Reader(['ch_sim', 'en'], gpu=True)

def recognize_with_easyocr(plate_img):
    """
    使用 EasyOCR 识别车牌
    支持中英文混合
    """
    results = reader.readtext(plate_img)
    
    for bbox, text, confidence in results:
        # 过滤非车牌字符
        cleaned = ''.join(c for c in text if c.isalnum())
        if len(cleaned) >= 6:  # 车牌至少6位
            return cleaned, confidence
    
    return '', 0.0
```

---

## 第六步：完整流水线

把所有步骤串联起来：

```python
import cv2
import numpy as np

class LicensePlateRecognizer:
    """车牌识别器——封装完整流程"""
    
    def __init__(self, templates_dir=None, use_easyocr=False):
        self.templates = None
        if templates_dir:
            self.templates = load_templates(templates_dir)
        self.use_easyocr = use_easyocr
        if use_easyocr:
            import easyocr
            self.reader = easyocr.Reader(['ch_sim', 'en'])
    
    def recognize(self, image_path):
        """
        完整的车牌识别流程
        输入：图像路径
        输出：{ plate_number, plate_image, location, confidences }
        """
        # Step 1: 预处理
        img, processed = preprocess(image_path)
        
        # Step 2: 车牌定位
        plate_rect = locate_plate(processed)
        if plate_rect is None:
            return {"error": "未检测到车牌"}
        
        # Step 3: 透视校正
        plate_img = correct_perspective(img, plate_rect)
        
        # Step 4: 字符识别
        plate_number = ''
        if self.use_easyocr:
            plate_number, confidence = recognize_with_easyocr(plate_img)
        else:
            # Step 4a: 字符分割
            chars = segment_characters(plate_img)
            # Step 4b: 模板匹配
            plate_number, confidences = recognize_plate(chars, self.templates)
        
        return {
            "plate_number": plate_number,
            "location": plate_rect,
            "plate_image": plate_img
        }


# 使用示例
if __name__ == "__main__":
    recognizer = LicensePlateRecognizer(
        templates_dir="./templates",
        use_easyocr=False
    )
    
    result = recognizer.recognize("./images/test_plate.jpg")
    
    if "error" not in result:
        print(f"识别结果: {result['plate_number']}")
        print(f"车牌位置: {result['location']}")
        
        # 显示结果
        cv2.imshow("Plate", result['plate_image'])
        cv2.waitKey(0)
```

## 测试与效果

### 测试用例设计

| 场景 | 挑战 | 解决策略 |
|------|------|---------|
| 正面光照良好 | 标准情况 | 基准流程即可 |
| 侧面倾斜拍摄 | 透视变形 | 透视校正步骤 |
| 阴天/傍晚 | 低对比度 | 直方图均衡化 |
| 车牌污损 | 字符残缺 | 形态学操作 + 降低匹配阈值 |
| 多车同框 | 多候选区域 | 按面积/位置筛选 |
| 夜间车灯 | 过曝 | 自适应阈值 |

### 性能分析

| 方案 | 准确率 | 速度 | 适用场景 |
|------|--------|------|---------|
| 模板匹配 | ~75% | < 0.5s | 标准字体、良好光照 |
| EasyOCR | ~92% | 1-2s | 复杂场景、多字体 |
| PaddleOCR | ~95% | 1-3s | 中文车牌、高精度需求 |

## 常见问题排查

### Q1: 定位不到车牌？

```
检查清单：
□ 图像分辨率是否足够（建议 800×600 以上）
□ 调整 Canny 阈值 (100, 200) → 试 (50, 150)
□ 调整形态学核大小 (17, 5) → 试 (15, 5) 或 (19, 5)
□ 放宽宽高比范围 (2.0~5.5) → 试 (1.5~6.0)
□ 减小面积阈值 2000 → 试 1000
```

### Q2: 识别结果有乱码？

```
检查清单：
□ 字符分割是否正确（查看分割结果可视化）
□ 模板库是否包含所有字符（特别是中文）
□ 提高匹配阈值（0.5 → 0.7）
□ 确保校正步骤正常工作
```

### Q3: 中文识别率低？

模板匹配对中文字符效果差，建议切换到 EasyOCR 或 PaddleOCR。

## 改进方向

| 方向 | 方案 | 难度 |
|------|------|------|
| 绿色新能源车牌 | 增加绿色区域检测 | ⭐⭐ |
| 多车牌同时识别 | NMS 去重 + 多区域处理 | ⭐⭐⭐ |
| 视频实时识别 | 帧间跟踪减少重复检测 | ⭐⭐⭐ |
| 端到端深度学习 | YOLO 定位 + CRNN 识别 | ⭐⭐⭐⭐ |
| 移动端部署 | 模型量化 + ncnn/TNN | ⭐⭐⭐⭐ |

## 总结

通过这个项目，你将掌握：

- ✅ OpenCV 图像处理全流程（读取→灰度化→模糊→边缘→形态学→轮廓）
- ✅ 计算机视觉核心算法（Canny、透视变换、模板匹配、投影法）
- ✅ 完整项目的工程化思路（模块化、Pipeline 设计、错误处理）
- ✅ 从传统方法到深度学习的演进路线

> "Talk is cheap. Show me the code." ——动手敲一遍，比看十遍更有效。

> 📂 完整代码：[GitHub - License_Plate_Recognition](https://github.com/TheShycute/License_Plate_Recognition)
