---
title: 【OCR】pdf文档版面分析——页眉、页脚、页码
date: 2026-06-11 15:34:23
tags:
    - OCR
    - 文档版面识别
---

这是Document Layout Analysis（文档版面分析），对于pdf扫描件，识别并剔除页眉、页脚与页码。

由于扫描件不存在文本层，无法通过PDF元数据获取，需要结合**OCR+版面分析+跨页统计**来实现

**整体流程：**
```
扫描PDF
   ↓
PDF转图片（300dpi）
   ↓
OCR识别（文字+坐标）
   ↓
版面分析（检测文本块）
   ↓
跨页统计分析
   ↓
识别页眉/页脚/页码
   ↓
清洗正文
```

## 为什么
在构建RAG向量时，纯OCR会导致以下错误：
- 页码会污染向量
- 页眉、页脚会降低召回质量

例如，不进行清洗，会得到如下文本：
```
版权所有XX公司

Transformer由Google提出……

第25页
```

正确的应该是：
```
Transformer由Google提出……
```

## 难点

单纯的OCR识别可能存在：
- OCR识别有误
- 页眉内容可能变化
- 页码每页不同
- 页脚可能带日期、水印

因此不能简单依赖固定文本匹配。

## 实现方案

> OCR + 坐标聚类 + 跨页重复检测

### OCR提取文字坐标

推荐：

工具|效果
|--|--|
PaddleOCR|中文最好
Tesseract OCR|免费
MinerU|PDF解析优秀
PP-StructureV3|版面分析

得到文本块：
```
[
    {
        "text": "第一章 概述",
        "bbox": [100. 500, 600, 560]
    }
]
```

### 坐标聚类（位置归一）

设置页面高度：
```
H = page_height
```

计算：
```
top_ratio = y1 / H
bottom_ratio = y2 / H
```

例如：
```
0~0.08       页眉区域
0.08~0.92    正文
0.92~1.0     页脚区域
```

转换：
```
for block in blocks:
    block["y_norm"] = block["bbox"][1] / H
```

### 跨页统计识别页眉
假设当前文档有100页

统计：
```
XX公司内部资料        95次
技术中心              94次
第一章 概述             5次
```

满足：
```
出现频率 > 70%
且
位置集中在顶部
```

判定：
```
header
```

示例：
```
Counter()

for page in pages:
    for block in page:
        if block["y_norm"] < 0.1:
            counter[normalize(block["text"])] += 1
```

判断：
```
count / total_pages > 0.7
```

### 识别页码

#### 页码特点：
- 位置固定：
    - 底部中间
    - 底部左侧
    - 底部右侧
    - 顶部右侧
- 数字规律
    - 常见模式：
        ```regex
        ^\d+$

        ^第\s*\d+\s*页$

        ^Page\s+\d+$

        ^\d+\s*/\s*\d+$

        ^共\d+页\s*第\d+页$
        ```

OCR后：
```
if re.match(pattern,text):
    candidate=True
```

#### 连续性验证

得到页码候选：
```
[1,2,3,5,6]
```
进行检查
```
页码差≈1
```
即可确认。



### 识别页脚
页脚特点：
- 位置接近底部
- 出现频率高
- 文本相似

例如：
```
Confidential
版权所有 XX公司
内部资料 请勿外传
```

统计：
```
版权所有 XX公司     100次
内部资料 请勿外传    99次
```

则判定其为页脚。


### 模糊匹配（解决OCR错误）
OCR识别出来的页眉页脚可能有误或者存在差异，使用`rapidfuzz`进行模糊匹配

例如：
```
from rapidfuzz import fuzz

similarity = fuzz.ratio(a,b)
```

若：
```
similarity > 90
```
则视为同一页眉。