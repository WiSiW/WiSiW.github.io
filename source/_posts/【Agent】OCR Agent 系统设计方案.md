---
title: 【Agent】OCR Agent 系统设计方案
date: 2026-10-30 17:22:14
tags:
    - Agent
    - OCR
---

# OCR Agent 系统设计方案：工具体系、Prompt 设计与具体实现

如果从 FDE（Forward Deployed Engineer）的角度设计一个 OCR Agent，我不建议简单地将 OCR 识别模型封装成一个 Tool，然后让大模型调用。

更合理的方案是：将 OCR Agent 设计成一个具备文档理解、任务规划、工具调用、质量评估、错误修复和结构化输出能力的智能文档处理 Agent。

它不仅需要识别图片中的文字，还需要理解文档结构，判断文档类型，识别表格、标题、段落、页眉页脚、印章、手写内容，并能够处理扫描件、PDF、复杂版式和多页文档。

例如，用户上传一份 PDF：

- 如果是普通 PDF，判断是否可以直接提取文本。
- 如果是扫描 PDF，调用 OCR 引擎识别。
- 如果包含表格，调用表格识别工具。
- 如果包含多栏排版，调用版面分析工具。
- 如果存在印章、手写文字，调用对应的专项识别工具。
- 如果识别结果置信度较低，自动进行图像增强、二次识别。
- 最后按照原始文档结构输出 Markdown、JSON、Excel 等结果。

下面按照实际工程落地的方式设计。

# 一、整体架构设计

## 1.1 核心架构

建议采用一个主 Agent + 多个专业 Tool 的架构。

![](./images/OCR-Agent.jfif)

整体分为六层：

| 层级      | 核心模块               | 主要职责                     |
| ------- | ------------------ | ------------------------ |
| 接入层     | Document Ingestion | 接收 PDF、图片、Word 等文件       |
| 感知层     | Image Processing   | 图片预处理、旋转矫正、去噪            |
| 分析层     | Layout Analysis    | 页面布局、文本区域、表格检测           |
| 识别层     | OCR Engine         | 文本检测、文本识别、公式及表格识别        |
| Agent 层 | OCR Agent          | 任务规划、工具路由、质量评估、重试        |
| 输出层     | Document Formatter | 输出 Markdown、JSON、Excel 等 |

## 1.2 Agent 内部结构

我建议将 OCR Agent 拆成以下 7 个核心模块：

OCR Main Agent

任务理解 · 规划 · Tool 调度 · 结果验证

1\. 文档分析模块

文件类型、页面、内容类型判断

2\. 图像预处理模块

旋转、去噪、增强、裁剪

3\. 版面分析模块

布局、阅读顺序、区域检测

4\. OCR 识别模块

文本、手写、公式识别

5\. 表格解析模块

表格结构、单元格、合并关系

6\. 质量评估模块

置信度、完整性、异常检测

7\. 结构化输出模块

Markdown、JSON、HTML、Excel、Word

Tool Registry / 工具注册中心

统一管理工具接口、参数 Schema、权限、超时、重试及执行日志

基础设施：对象存储 · 消息队列 · 数据库 · OCR 模型服务 · 日志监控

这里需要注意：

Agent 负责决策，Tool 负责执行，OCR 模型负责识别，质量评估模块负责判断结果是否可信。

不要让 LLM 直接承担文字识别任务。文字识别应当交给专用 OCR 模型，LLM 更适合处理文档理解、任务规划、结果校验和结构化组织。

## 二、Tool 工具体系设计

这是整个 OCR Agent 最重要的部分。

我建议将工具划分为 5 大类，共设计 18 个核心 Tool。每个 Tool 都有明确的输入参数、输出结构和职责边界。

### 2.1 文档接入类 Tool

| Tool                 | 功能                  | 输入      | 输出            |
| -------------------- | ------------------- | ------- | ------------- |
| `document_loader`    | 加载 PDF、图片、Office 文件 | 文件路径    | 文档对象          |
| `document_analyzer`  | 分析文件类型和文档属性         | 文档对象    | 文件类型、页数、是否扫描件 |
| `pdf_text_extractor` | 提取 PDF 原生文本         | PDF、页码  | 文本及坐标         |
| `pdf_to_image`       | PDF 页面转图片           | PDF、DPI | 页面图片          |

这里有一个重要设计：

先判断 PDF 是否有原生文本层。如果可以直接提取，就不要无条件进行 OCR。

例如：

```
{
  "document_type": "pdf",
  "page_count": 20,
  "native_text_ratio": 0.92,
  "is_scanned": false,
  "recommended_strategy": "native_text_extraction"
}
```

对于混合 PDF，还需要支持逐页判断：

- 第 1 页：原生文本。
- 第 2 页：扫描图片。
- 第 3 页：原生文本 + 图片。
- 第 4 页：扫描图片。

这样可以避免对整个 PDF 重复执行 OCR。

### 2.2 图像预处理类 Tool

| Tool             | 功能       | 主要实现            |
| ---------------- | -------- | --------------- |
| `image_denoise`  | 图像去噪     | OpenCV          |
| `image_binarize` | 二值化处理    | OpenCV          |
| `image_deskew`   | 文档倾斜矫正   | OpenCV          |
| `image_rotate`   | 方向检测与旋转  | OpenCV / OCR 模型 |
| `image_enhance`  | 对比度、锐度增强 | OpenCV          |
| `image_crop`     | 根据坐标裁剪区域 | Pillow / OpenCV |

这里不要让 Agent 每次都调用所有预处理工具。

例如：

- 清晰图片：直接 OCR。
- 图片倾斜：调用 `image_deskew`。
- 图片模糊：调用 `image_enhance`。
- 背景噪声严重：调用 `image_denoise`。
- 局部文字识别失败：调用 `image_crop`，对局部区域重新识别。

### 2.3 OCR 识别类 Tool

| Tool                      | 功能      | 推荐技术           |
| ------------------------- | ------- | -------------- |
| `text_detection`          | 检测文本区域  | PaddleOCR      |
| `text_recognition`        | 识别文本内容  | PaddleOCR      |
| `layout_analysis`         | 文档版面分析  | PP-StructureV3 |
| `table_recognition`       | 表格识别    | PP-StructureV3 |
| `formula_recognition`     | 数学公式识别  | 公式识别模型         |
| `seal_recognition`        | 印章检测与识别 | 印章识别模型         |
| `handwriting_recognition` | 手写文字识别  | 手写专用模型         |

其中 `layout_analysis` 非常关键。

普通 OCR 的输出通常只是：

```
{
  "text": "患者基本信息",
  "confidence": 0.98
}
```

但对于复杂文档，我们需要得到：

```
{
  "page": 1,
  "elements": [
    {
      "type": "title",
      "text": "患者基本信息",
      "bbox": [100, 50, 500, 100],
      "confidence": 0.99
    },
    {
      "type": "paragraph",
      "text": "姓名：张某某",
      "bbox": [100, 120, 500, 160],
      "confidence": 0.97
    },
    {
      "type": "table",
      "bbox": [100, 200, 900, 600],
      "confidence": 0.96
    }
  ]
}
```

这样才能还原文档的组织结构。

实际工程中，可以直接使用 PaddleOCR 的 PP-StructureV3。它提供版面检测、OCR、表格、公式、印章等子流程，并支持阅读顺序恢复和 Markdown 输出。

[image](https://www.google.com/s2/favicons?domain=https://www.paddleocr.ai\&sz=32)

PaddleOCR Documentation+1



### 2.4 质量评估类 Tool

这一类工具决定 OCR Agent 是否真正具备自主纠错能力。

| Tool                     | 功能      | 判断依据          |
| ------------------------ | ------- | ------------- |
| `ocr_quality_check`      | 识别质量评估  | OCR 置信度、文本完整性 |
| `text_consistency_check` | 文本一致性检查 | 字段格式、上下文      |
| `layout_integrity_check` | 布局完整性检查 | 区域覆盖、阅读顺序     |
| `ocr_error_detector`     | 识别异常检测  | 乱码、缺字、异常字符    |

例如，识别结果如下：

```
{
  "text": "患者年龄：1S岁",
  "confidence": 0.62
}
```

质量检测 Tool 发现：

- 置信度较低。
- 年龄字段存在异常字符。
- 数字与字母混淆。
- 当前区域可能需要二次识别。

然后返回：

```
{
  "status": "needs_retry",
  "issues": [
    {
      "type": "low_confidence",
      "field": "患者年龄",
      "confidence": 0.62
    },
    {
      "type": "character_ambiguity",
      "expected_type": "number",
      "actual_value": "1S"
    }
  ],
  "suggested_action": "crop_and_recognize"
}
```

注意：不能直接根据上下文把 `1S` 自动改成 `18`。上下文可以帮助发现异常，但必须通过原始图像二次识别或人工确认，避免篡改原始信息。

### 2.5 结果输出类 Tool

| Tool                | 功能            | 输出      |
| ------------------- | ------------- | ------- |
| `markdown_export`   | Markdown 文档生成 | `.md`   |
| `json_export`       | 结构化 JSON      | `.json` |
| `excel_export`      | 表格数据导出        | `.xlsx` |
| `word_export`       | Word 文档生成     | `.docx` |
| `ocr_visualization` | OCR 区域可视化     | 标注图片    |

## 三、Tool Registry 工具注册中心

实际开发中，不建议把 18 个 Tool 的代码直接写在 Agent 主逻辑中。

应该设计一个统一的工具注册中心。

### 3.1 工具接口规范

所有工具统一使用以下接口：

```
from pydantic import BaseModelfrom typing import Any, Dict, Optionalclass ToolResult(BaseModel):    success: bool    tool_name: str    data: Optional[Dict[str, Any]] = None    error: Optional[str] = None    execution_time: float = 0    metadata: Dict[str, Any] = {}class BaseTool:    name: str = ""    description: str = ""    def execute(self, **kwargs) -> ToolResult:        raise NotImplementedError
```

### 3.2 注册中心实现

```
class ToolRegistry:    def __init__(self):        self.tools = {}    def register(self, tool):        self.tools[tool.name] = tool    def get_tool(self, name):        if name not in self.tools:            raise ValueError(f"Tool not found: {name}")        return self.tools[name]    def list_tools(self):        return [            {                "name": tool.name,                "description": tool.description            }            for tool in self.tools.values()        ]    def execute(self, name, **kwargs):        tool = self.get_tool(name)        return tool.execute(**kwargs)
```

### 3.3 Tool Registry 应该管理哪些信息？

除了工具名称，还应该增加：

| 属性            | 说明          |
| ------------- | ----------- |
| name          | 工具名称        |
| description   | 工具描述        |
| input_schema  | 输入参数 Schema |
| output_schema | 输出参数 Schema |
| timeout       | 执行超时时间      |
| retry_policy  | 重试策略        |
| permission    | 工具权限        |
| idempotent    | 是否支持幂等执行    |
| enabled       | 是否启用        |
| version       | 工具版本        |

例如：

```
OCR_TOOL_CONFIG = {    "text_recognition": {        "timeout": 30,        "max_retries": 2,        "permission": "document:read",        "idempotent": True,        "version": "1.0.0"    },    "table_recognition": {        "timeout": 60,        "max_retries": 2,        "permission": "document:read",        "idempotent": True,        "version": "1.0.0"    }}
```

这里建议区分三类操作：

1. 只读工具：文本提取、图片分析、OCR 识别。
2. 计算工具：图片增强、表格解析、质量评估。
3. 写入工具：文件导出、结果保存。

写入工具必须增加权限控制、任务隔离和输出路径校验。

# 四、OCR Agent 的 Prompt 如何设计？

Prompt 不应该只有一个。

我建议设计成：

System Prompt + Planning Prompt + Tool Description + Quality Evaluation Prompt + Output Prompt。

这样可以将任务规划、工具执行、质量判断和结果组织分开，便于后续维护。

## 4.1 System Prompt：主 Agent 提示词

这是 OCR Agent 最核心的 Prompt。

# Role：OCR Document Processing Agent

你是一名专业的 OCR 文档智能处理 Agent，负责处理 PDF、图片及其他文档格式。

你的目标不是简单地提取文字，而是尽可能完整、准确地恢复文档中的文本内容、版面结构和业务信息。

## 一、核心职责

1. 分析用户上传的文档，判断文档格式、页数及内容类型。
2. 根据文档特征选择合适的处理策略。
3. 通过 Tool 调用完成文件解析、图片预处理、OCR 识别、表格识别及版面分析。
4. 对 OCR 结果进行质量评估。
5. 对低置信度、缺失内容及异常区域执行二次识别。
6. 按照用户要求输出 Markdown、JSON、Excel 等结构化结果。

## 二、任务执行原则

1. 优先提取 PDF 原生文本，避免无意义的重复 OCR。
2. 对扫描件执行图像预处理及 OCR。
3. 对复杂排版文档执行版面分析。
4. 对表格、公式、印章、手写内容使用专项识别工具。
5. 不允许将模型推测的内容当作 OCR 原始识别结果。
6. 不允许擅自修改原始识别内容。
7. 对无法确认的内容使用 uncertain 标记。
8. 保留原始页面、区域坐标及置信度。
9. 所有工具调用必须使用已注册的工具。
10. 不得伪造工具执行结果。

## 三、任务规划

根据文档类型选择策略：

- 普通 PDF：pdf_text_extractor。
- 扫描 PDF：pdf_to_image + OCR。
- 混合 PDF：逐页判断，分别处理。
- 图片：图像分析 + 必要的预处理 + OCR。
- 复杂排版：layout_analysis。
- 表格：table_recognition。
- 公式：formula_recognition。
- 印章：seal_recognition。
- 手写内容：handwriting_recognition。

## 四、质量评估

重点检查：

- OCR 识别置信度。
- 文本区域覆盖率。
- 页面内容完整性。
- 表格行列完整性。
- 阅读顺序正确性。
- 乱码及异常字符。
- 多页文档的页码连续性。

发现问题时，根据异常类型选择合适的修复工具。

## 五、重试策略

1. 低置信度：重新裁剪并识别。
2. 图片模糊：图像增强后重新识别。
3. 图片倾斜：矫正后重新识别。
4. 表格异常：重新调用表格识别。
5. 版面异常：重新执行布局分析。
6. 多次识别仍失败：标记为人工复核。

每个区域最多执行两次自动重试，避免无限循环。

## 六、输出要求

必须区分：

- original_text：原始 OCR 识别结果。
- normalized_text：经过规则标准化的结果。
- confidence：识别置信度。
- bbox：原始页面坐标。
- status：识别状态。
- issues：异常信息。

最终输出必须忠实于原始文档，不得将推测内容混入识别结果。

## 4.2 Planning Prompt：任务规划提示词

主 Agent 接收到文件后，先不直接执行 OCR，而是生成任务计划。

例如用户上传一份 50 页的 PDF：

# Role：OCR Task Planner

你负责分析文档，并生成 OCR 执行计划。

根据以下信息制定处理策略：

输入信息：

- 文件类型：{{file_type}}
- 页面数量：{{page_count}}
- 原生文本比例：{{native_text_ratio}}
- 页面图像质量：{{image_quality}}
- 文档类型：{{document_type}}
- 用户需求：{{user_requirement}}

请完成：

1. 判断文档是否需要 OCR。
2. 判断是否需要逐页处理。
3. 判断是否需要版面分析。
4. 判断是否包含表格、公式、印章及手写内容。
5. 判断是否需要图像预处理。
6. 指定所需工具及执行顺序。
7. 设置质量评估标准。

输出严格 JSON：

{\
"strategy": "hybrid",\
"tasks": [\
{\
"page": 1,\
"task_type": "native_text_extraction",\
"tools": ["pdf_text_extractor"],\
"priority": 1\
},\
{\
"page": 2,\
"task_type": "ocr",\
"tools": [\
"image_deskew",\
"layout_analysis",\
"text_recognition"\
],\
"priority": 2\
}\
],\
"quality_threshold": 0.90,\
"max_retries": 2,\
"output_format": "markdown"\
}

注意：\
只能选择工具注册表中真实存在的工具，不得生成不存在的工具名称。

这里的 `quality_threshold` 只是任务策略的初始配置，不代表 OCR 单字置信度达到 0.90 就能保证文档正确。具体阈值应当通过测试数据校准。

## 4.3 Quality Evaluation Prompt：质量评估提示词

这个 Prompt 主要负责发现 OCR 识别异常。

# Role：OCR Quality Evaluator

你是一名 OCR 结果质量评估专家。

输入：

- 原始页面图片：{{page_image}}
- OCR 识别结果：{{ocr_result}}
- 页面布局：{{layout_result}}
- 识别置信度：{{confidence}}

请检查：

1. 是否存在明显的漏字、错字。
2. 是否存在乱码。
3. 是否存在数字与字母混淆。
4. 是否存在表格行列错位。
5. 是否存在阅读顺序错误。
6. 是否存在文本区域遗漏。
7. 是否存在页眉、页脚或页码遗漏。

对于每一个问题，返回：

- 问题类型。
- 所属页面。
- 对应文本区域。
- 问题描述。
- 严重程度。
- 建议采取的修复工具。

特别约束：

1. 必须以原始图像为依据。
2. 不允许仅凭语义合理性修改 OCR 结果。
3. 不允许将推测内容标记为已确认内容。
4. 无法确认的内容必须标记 uncertain。
5. 对无法通过自动识别解决的问题，建议人工复核。

输出 JSON：
```
{
    "quality_status": "needs_retry",
    "issues": [
        {
            "type": "character_error",
            "page": 2,
            "bbox": [100, 200, 300, 250],
            "original_text": "1S岁",
            "severity": "medium",
            "suggested_tool": "image_crop"
        }
    ],
    "recommendation": "crop_and_recognize"
}
```

一个重要的工程原则： 质量评估不能只依赖 LLM 对 OCR 文本进行语义判断。应该综合 OCR 置信度、图像区域覆盖率、字符规则校验、版面结构校验和必要的人工抽检。

## 4.4 Output Prompt：结果组织提示词

# Role：Document Formatter

你负责将 OCR 结果整理成用户要求的文档格式。

输入：

- OCR 原始结果：{{ocr_result}}
- 版面结构：{{layout_result}}
- 表格结果：{{table_result}}
- 用户指定格式：{{output_format}}

要求：

1. 保持原始文档阅读顺序。
2. 保留标题、段落、列表及表格结构。
3. 保留页码。
4. 表格不得随意改变行列关系。
5. 保留无法识别的内容标记。
6. 不允许补充原始文档中不存在的信息。
7. 不允许将推测内容写入原始 OCR 字段。

根据用户要求生成 Markdown、JSON 或其他格式。

# 五、核心实现：Python + PaddleOCR + LangGraph

接下来给出可以作为实际项目基础的代码。

技术选型：

| 技术             | 作用          |
| -------------- | ----------- |
| Python 3.11    | 后端开发        |
| FastAPI        | OCR 服务接口    |
| PaddleOCR 3.x  | OCR 识别引擎    |
| PP-StructureV3 | 文档版面分析      |
| OpenCV         | 图像预处理       |
| PyMuPDF        | PDF 解析与页面转换 |
| Pydantic       | Tool 参数校验   |
| LangGraph      | Agent 流程编排  |
| Redis          | 任务状态、缓存     |
| MinIO          | 原始文件及结果存储   |

这里推荐使用 LangGraph 实现 Agent 工作流，而不是完全依赖一个自由循环的 ReAct Agent。

原因是 OCR 是一个具有明确执行顺序、质量门槛和重试条件的任务，使用状态图更容易控制执行过程。

## 5.1 项目目录结构

```
ocr-agent/
│
├── app/
│   ├── main.py
│   │
│   ├── agent/
│   │   ├── ocr_agent.py
│   │   ├── planner.py
│   │   ├── evaluator.py
│   │   ├── prompts.py
│   │   └── workflow.py
│   │
│   ├── tools/
│   │   ├── base.py
│   │   ├── registry.py
│   │   ├── document_tools.py
│   │   ├── image_tools.py
│   │   ├── ocr_tools.py
│   │   ├── layout_tools.py
│   │   ├── quality_tools.py
│   │   └── export_tools.py
│   │
│   ├── services/
│   │   ├── pdf_service.py
│   │   ├── paddle_service.py
│   │   └── storage_service.py
│   │
│   ├── schemas/
│   │   ├── document.py
│   │   ├── ocr.py
│   │   └── result.py
│   │
│   └── config/
│       └── settings.py
│
├── storage/
├── tests/
├── requirements.txt
└── README.md
```

## 5.2 安装依赖

```
fastapi
uvicorn
python-multipart
pydantic
paddleocr
paddlepaddle
opencv-python-headless
pymupdf
pillow
numpy
langgraph
langchain
langchain-openai
```

安装 PaddleOCR 后，首次启动还需要准备对应模型文件。生产环境建议预先下载模型并固定版本，避免服务启动时临时下载。

## 5.3 OCR 引擎封装

这里直接使用 PP-StructureV3，将版面分析、文本识别和表格解析整合起来。

`app/services/paddle_service.py`

```
import jsonfrom paddleocr import PPStructureV3class PaddleOCRService:    def __init__(self, device="gpu"):        self.pipeline = PPStructureV3(            device=device,            use_doc_orientation_classify=True,            use_doc_unwarping=True,            use_textline_orientation=True        )    def predict(self, image_path):        outputs = self.pipeline.predict(image_path)        results = []        for output in outputs:            data = output.json            if callable(data):                data = data()            if isinstance(data, str):                data = json.loads(data)            results.append(data)        return results
```

这里封装的是一个统一的文档解析引擎。不同模型版本的输出字段可能存在变化，因此需要在服务层统一适配，不能让 Agent 直接依赖 PaddleOCR 的原始返回结构。

## 5.4 图像预处理 Tool

`app/tools/image_tools.py`

```
import cv2import numpy as npclass ImageTools:    @staticmethod    def enhance(image_path, output_path):        image = cv2.imread(image_path)        if image is None:            raise ValueError("Invalid image")        # 灰度化        gray = cv2.cvtColor(            image,            cv2.COLOR_BGR2GRAY        )        # CLAHE 对比度增强        clahe = cv2.createCLAHE(            clipLimit=2.0,            tileGridSize=(8, 8)        )        enhanced = clahe.apply(gray)        # 降噪        denoised = cv2.fastNlMeansDenoising(            enhanced,            None,            10,            7,            21        )
```

注意：这只是基础预处理实现。生产环境还应该根据文档类型选择不同算法。特别是复杂表格、证件和弯曲书页，不能仅靠简单的倾斜矫正。

## 5.5 PDF 解析 Tool

`app/tools/document_tools.py`

```
import fitzclass PDFTools:    @staticmethod    def analyze(pdf_path):        doc = fitz.open(pdf_path)        results = []        for index, page in enumerate(doc):            text = page.get_text("text")            results.append({                "page": index + 1,                "text": text,                "has_native_text": bool(                    text.strip()                ),                "text_length": len(text)            })        return {            "page_count": len(doc),            "pages": results        }    @staticmethod    def to_images(        pdf_path,        output_dir,        dpi=200    ):
```

这里可以进一步增加原生文本覆盖率计算：

```
def calculate_text_ratio(text, page_area):    """    实际项目中结合文本块面积、    页面有效区域及扫描图像面积计算。    """    if not text.strip():        return 0.0    return min(len(text.strip()) / 500, 1.0)
```

这个函数仅是演示占位逻辑。真实的文本覆盖率应当结合页面文本块面积、字符数量、图片覆盖区域等计算，不能直接用字符数除以一个固定值作为准确指标。

## 5.6 OCR Tool 封装

`app/tools/ocr_tools.py`

```
import timefrom app.services.paddle_service import (    PaddleOCRService)class OCRRecognitionTool:    name = "text_recognition"    description = """    对文档图片执行 OCR 识别，    返回文本、版面结构、表格及置信度。    """    def __init__(self):        self.service = PaddleOCRService(            device="gpu"        )    def execute(self, image_path):        start = time.time()        try:            results = self.service.predict(                image_path            )            return {                "success": True,                "data": results,                "execution_time": (
```

## 5.7 OCR 质量评估 Tool

这是我建议重点实现的模块。

`app/tools/quality_tools.py`

```
import reclass OCRQualityTool:    def execute(self, ocr_result):        issues = []        texts = []        scores = []        def collect(data):            if isinstance(data, dict):                for key, value in data.items():                    if key in (                        "rec_texts",                        "rec_scores"                    ):                        if isinstance(value, list):                            if key == "rec_texts":                                texts.extend(value)                            else:                                scores.extend(value)                    else:                        collect(value)            elif isinstance(data, list):                for item in data:                    collect(item)
```

这里有一个需要特别注意的地方：不同版本的 PP-StructureV3 返回数据结构不同，实际使用时应当先将原始结果转换成统一的 `OCRResult` 数据结构，再进行质量评估。上面的代码展示了核心检查逻辑，不应直接当作所有模型版本通用的解析器。

# 六、Agent 工作流：如何实现自主决策？

前面实现的是工具，现在要把工具连接起来。

我建议采用一个具有状态管理能力的工作流。

## 6.1 工作流设计

[Rethinking Document Processing Pipelines](https://images.openai.com/static-rsc-4/ykdwHIfjtjmYGe9AlioxIYV4uC5UHxmUxamcVrTkUuz2OxNmZBNGaRVtK1shdmHx-ZUjvrPmPHghNh4799LnyRwyI6nK2k1DOc-8bydBlGzkgkaq0pWp4tC42Pvgu-N8VXsFOic5hkLRoqrcpNPKInuGSZwrXNu8z9Zwi99zOSc?purpose=inline)

[aloa.co](https://aloa.co/ai/resources/deep-dive/rethinking-document-processing-pipelines?utm_source=chatgpt.com)

核心流程：

1. 接收文件。
2. 分析文档类型。
3. 制定 OCR 任务计划。
4. 执行原生文本提取或 OCR。
5. 进行版面及专项内容解析。
6. 质量评估。
7. 如果不合格，进入纠错流程。
8. 重新识别。
9. 生成最终文档。

其中质量评估节点是一个条件分支：

```
              OCR 识别
                  |
                  v
              质量评估
                  |
          +-------+-------+
          |               |
        合格            不合格
          |               |
          v               v
       结果输出        错误分类
                          |
                  +-------+-------+
                  |       |       |
                模糊    倾斜    低置信度
                  |       |       |
                  v       v       v
                增强    矫正    局部裁剪
                  |       |       |
                  +-------+-------+
                          |
                       二次 OCR
                          |
                       再次评估
                          |
                   达到重试上限？
                      /      \
                    是        否
                    |          |
                 人工复核    继续重试
```

## 6.2 LangGraph State 设计

`app/agent/workflow.py`

```
from typing import TypedDict, Any, Listclass OCRState(TypedDict):    file_path: str    document_info: dict    task_plan: dict    page_images: List[str]    ocr_results: List[dict]    quality_results: dict    retry_count: int    max_retries: int    final_result: Any    status: str
```

## 6.3 定义 Agent 节点

```
from langgraph.graph import (    for result in state["ocr_results"]:        if result["success"]:            quality = quality_tool.execute(                result["data"]            )            results.append(quality)    passed = all(        item["status"] == "passed"        for item in results    )    return {        "quality_results": {            "passed": passed,            "details": results        }    }def retry_node(state):    return {        "retry_count": (            state["retry_count"] + 1        ),        "status": "retrying"    }def export_node(state):    return {        "final_result": {            "ocr_results": state["ocr_results"],            "quality": state["quality_results"]        },        "status": "completed"    }def route_quality(state):    if state["quality_results"]["passed"]:        return "export"    if state["retry_count"] >= state["max_retries"]:        return "export"    return "retry"
```

## 6.4 构建工作流

```
def build_ocr_graph():    graph.add_node(        "ocr",        ocr_node    )    graph.add_node(        "quality",        quality_node    )    graph.add_node(        "retry",        retry_node    )    graph.add_node(        "export",        export_node    )    graph.add_edge(START, "analyze")    graph.add_edge(        "analyze",        "planning"    )    graph.add_edge(        "planning",        "ocr"    )    graph.add_edge(        "ocr",        "quality"    )    graph.add_conditional_edges(        "quality",        route_quality,        {            "export": "export",            "retry": "retry"        }    )    graph.add_edge(        "retry",        "ocr"    )    graph.add_edge(        "export",        END    )    return graph.compile()
```

以上是工作流骨架，距离生产环境还有两个关键完善点：

- 重试节点需要根据异常类型选择不同的修复操作，而不是无条件重复 OCR。
- 对达到重试上限的页面，需要标记 `manual_review`，不能直接将不合格结果当作正常完成。

此外，生产环境还应当使用 LangGraph Checkpointer 保存状态，避免长文档处理过程中服务重启导致任务丢失。

# 七、如何实现真正的动态 Tool 路由？

上面的工作流属于固定流程，而一个成熟的 OCR Agent 应该支持动态路由。

例如：

用户上传的是一张身份证照片，Agent 不需要调用表格识别、公式识别。

用户上传的是财务报表，Agent 应该重点调用表格识别。

用户上传的是扫描书籍，Agent 应该重点关注多页 OCR、双栏阅读顺序、页眉页脚和章节结构。

## 7.1 动态路由规则

建议将文档分析结果转化为路由条件：

| 文档特征       | 路由策略          |
| ---------- | ------------- |
| PDF 原生文本完整 | PDF 原生文本提取    |
| PDF 无文本层   | PDF 转图片 + OCR |
| 图片倾斜       | 方向检测 + 倾斜矫正   |
| 图片模糊       | 图像增强 + OCR    |
| 包含表格       | OCR + 表格识别    |
| 包含数学公式     | OCR + 公式识别    |
| 包含印章       | 印章识别          |
| 包含手写文字     | 手写识别          |
| 多栏文档       | 版面分析 + 阅读顺序恢复 |

## 7.2 动态路由实现

可以让 LLM 根据文档分析结果生成执行计划，但最终工具执行由后端进行 Schema 校验。

例如：

```
def route_document(document_info):    routes = []    if document_info["is_scanned"]:        routes.append("text_recognition")    else:        routes.append("pdf_text_extractor")    if document_info.get("has_table"):        routes.append("table_recognition")    if document_info.get("has_formula"):        routes.append("formula_recognition")    if document_info.get("has_seal"):        routes.append("seal_recognition")    if document_info.get("complex_layout"):        routes.append("layout_analysis")    return routes
```

这里的 `has_table`、`has_formula` 等属性，应当由版面检测模型或文档分析模型提供，而不是凭空判断。

进一步可以将路由升级为：

规则引擎 + LLM Planner + Tool Registry。

- 规则引擎：负责确定性判断，例如文件格式、页数、是否存在原生文本。
- LLM Planner：负责复杂场景下的任务规划。
- Tool Registry：负责验证工具是否存在、参数是否合法。
- Workflow Engine：负责执行、重试和状态管理。

这样既保留 Agent 的灵活性，又避免 LLM 随意调用工具。

# 八、FastAPI 接口实现

最后将 Agent 封装为一个服务。

`app/main.py`

```
from fastapi import FastAPI, UploadFile, Filefrom pathlib import Pathimport shutilimport uuidapp = FastAPI(    title="OCR Agent API")ocr_graph = build_ocr_graph()@app.post("/api/ocr/parse")async def parse_document(    file: UploadFile = File(...)):    task_id = str(uuid.uuid4())    storage_dir = Path("./storage")    storage_dir.mkdir(        parents=True,        exist_ok=True    )    file_path = (        storage_dir /        f"{task_id}_{Path(file.filename).name}"    )    with open(file_path, "wb") as f:        shutil.copyfileobj(            file.file,            f        )
```

调用方式：

```
curl -X POST \
  http://localhost:8000/api/ocr/parse \
  -F "file=@test.pdf"
```

注意：以上 FastAPI 代码是同步工作流的最小演示版本。正式部署应增加文件大小和格式校验、异步任务队列、任务状态查询、用户权限、临时文件清理，以及独立 OCR 推理服务。对于长文档，不建议在 HTTP 请求中同步等待所有页面识别完成。

# 九、生产环境需要重点解决的 6 个问题

前面的实现解决了基本 OCR 流程，但如果真正用于企业级文档处理，我认为还需要重点考虑下面六个方面。

## 9.1 多页文档并行处理

假设一份 PDF 有 500 页。

不能简单地让 Agent 按照：

```
第1页 → 第2页 → 第3页 → ... → 第500页
```

串行执行。

建议：

```
             PDF 文件
                 |
             页面拆分
                 |
          Kafka 任务队列
                 |
       +---------+---------+
       |         |         |
     Worker1   Worker2   Worker3
       |         |         |
     OCR       OCR       OCR
       |         |         |
       +---------+---------+
                 |
             结果聚合
                 |
             阅读顺序恢复
                 |
              质量评估
                 |
               导出
```

需要注意：

- 页面任务独立分配。
- 通过 `document_id + page_number` 保证任务唯一性。
- 支持失败重试。
- 通过 Checkpoint 保存完成进度。
- 结果按照原始页码排序。
- 控制 GPU 并发，避免显存耗尽。

## 9.2 OCR 结果的数据结构统一

我建议不要直接保存 OCR 模型原始 JSON，而是建立统一的数据结构。

例如：

```
{
  "document_id": "doc_001",
  "pages": [
    {
      "page_number": 1,
      "width": 2480,
      "height": 3508,
      "elements": [
        {
          "element_id": "e001",
          "type": "paragraph",
          "bbox": [100, 100, 800, 200],
          "original_text": "患者基本信息",
          "normalized_text": "患者基本信息",
          "confidence": 0.98,
          "status": "verified"
        },
        {
          "element_id": "e002",
          "type": "table",
          "bbox": [100, 300, 1200, 900],
          "content": {
            "rows": 5,
            "columns": 4
          },
          "confidence": 0.96,
          "status": "verified"
        }
      ]
    }
  ]
}
```

建议将状态定义为：

- `verified`：通过质量校验。
- `low_confidence`：置信度不足。
- `needs_retry`：等待二次识别。
- `manual_review`：需要人工确认。
- `failed`：识别失败。

特别建议保留原始图像坐标，后续可以实现 OCR 结果与原图的区域定位、点击查看和人工修正。

## 9.3 人工反馈闭环

对于企业级 OCR，人工校对不是失败，而是提升识别质量的重要数据来源。

建议实现：

```
OCR 识别
   |
质量评估
   |
低质量区域
   |
人工校对
   |
修正后的文本
   |
保存标注数据
   |
构建训练数据集
   |
模型评估与微调
   |
模型版本更新
```

通过不断积累真实业务文档，可以针对特定场景优化 OCR 模型。

例如：

- 医疗病例。
- 财务报表。
- 合同。
- 发票。
- 手写病历。
- 历史书籍。

## 9.4 文档安全和权限

建议实现：

1. 上传文件进行病毒扫描和文件格式校验。
2. 每个用户只能访问授权文档。
3. 文件存储采用用户及租户隔离。
4. OCR 结果继承原始文档权限。
5. 对敏感字段进行脱敏处理。
6. 文件设置生命周期，支持到期清理。
7. 记录文件访问、识别、导出操作日志。

## 9.5 OCR 评估体系

不能只使用平均置信度作为 OCR Agent 的质量指标。

建议建立以下评估体系：

| 指标                     | 含义        |
| ---------------------- | --------- |
| CER                    | 字符错误率     |
| WER                    | 词错误率      |
| Detection Recall       | 文本区域检测召回率 |
| Layout mAP             | 版面检测准确性   |
| Table TEDS             | 表格结构相似度   |
| Reading Order Accuracy | 阅读顺序准确率   |
| Retry Rate             | 二次识别比例    |
| Manual Review Rate     | 人工复核比例    |
| Processing Latency     | 平均处理耗时    |
| Cost Per Page          | 单页识别成本    |

其中：

```
CER=(S+D+I)/N
```

- S：替换字符数。
- D：删除字符数。
- I：插入字符数。
- N：标准文本字符总数。

例如标准文本为 1000 个字符，识别出现 20 个错误字符，则 CER 为 2%。

## 9.6 断点续传与任务恢复

对于大型 PDF，必须支持断点续传。

例如：

```
{
  "document_id": "doc_001",
  "total_pages": 500,
  "completed_pages": 350,
  "failed_pages": [12, 45, 102],
  "pending_pages": 147,
  "status": "processing"
}
```

如果 OCR 服务重启，只需要重新执行失败页面和未完成页面，不需要重新识别全部 500 页。

# 十、最终推荐：OCR Agent 的版本迭代路线

如果让我真正负责这个项目，我不会一开始就开发全部 18 个 Tool，而是分阶段交付。

V1.0 基础 OCR Agent

优先实现

- PDF、图片上传
- PDF 原生文本提取
- PaddleOCR 接入
- 图片预处理
- 基础质量评估
- Markdown、JSON 导出

目标：实现完整的文档识别闭环。

V1.1 智能文档解析 Agent

- PP-StructureV3 版面分析
- 表格识别
- 公式识别
- 印章识别
- 多栏阅读顺序恢复
- 动态 Tool 路由

目标：从文字识别升级到文档结构理解。

V1.2 自主纠错 Agent

- 质量评估
- 低置信度区域检测
- 局部裁剪与二次识别
- 图像增强策略
- 人工校对
- 模型反馈数据集

目标：具备自动发现问题和修复问题的能力。

V1.3 企业级 OCR Agent

- 多页并行处理
- Kafka 任务调度
- Redis 状态管理
- MinIO 文件存储
- 多租户权限
- Checkpoint
- 监控与成本分析

目标：支持大规模文档处理和生产部署。

# 十一、我的最终架构建议

我认为这个项目最值得关注的不是 OCR 模型本身，而是三个核心能力：

第一，智能路由能力。

能够根据文件类型、图像质量、文档版式及用户需求，动态选择最合适的识别工具，而不是所有文档走同一套 OCR 流程。

第二，自主纠错能力。

能够根据置信度、文本完整性、版面结构和规则校验发现识别问题，进一步选择图像增强、区域裁剪、二次识别等操作。

第三，文档结构理解能力。

不仅提取文字，还要理解标题、正文、表格、图片、公式、页眉页脚及其相互关系，最终还原成可供 RAG、知识库和下游 Agent 使用的结构化文档。

技术上，我推荐：

- OCR 引擎：PaddleOCR 3.x + PP-StructureV3。
- Agent 编排：LangGraph。
- 图像处理：OpenCV。
- 文件处理：PyMuPDF。
- 消息队列：Kafka。
- 状态管理：Redis + PostgreSQL。
- 文件存储：MinIO。

官方技术文档：

- [PaddleOCR PP-StructureV3 使用文档](https://www.paddleocr.ai/latest/en/version3.x/pipeline_usage/PP-StructureV3.html?utm_source=chatgpt.com)
- [PP-StructureV3 算法介绍](https://paddlepaddle.github.io/PaddleOCR/main/en/version3.x/algorithm/PP-StructureV3/PP-StructureV3.html?utm_source=chatgpt.com)
- [PP-StructureV3 输出结构说明](https://www.paddleocr.ai/v3.0.3/en/version3.x/pipeline_usage/PP-StructureV3.html?utm_source=chatgpt.com)

还有一个值得进一步扩展的方向： 将 OCR Agent 与 RAG Agent 结合。OCR Agent 负责将 PDF、扫描件、图片转化为带有页码、坐标、标题层级、表格结构和来源信息的文档对象，RAG Agent 再负责切片、Embedding、索引和检索。这样就可以形成一套完整的企业级智能文档理解系统。