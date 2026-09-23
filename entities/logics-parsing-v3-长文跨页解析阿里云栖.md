---
title: "Logics-Parsing-V3：循环滑动窗口长文解析"
created: 2026-09-23
updated: 2026-09-23
type: entity
tags: [model, document-parsing, multimodal, ocr, alibaba, open-source]
sources: [raw/articles/logics-parsing-v3-长文跨页解析阿里云栖]
confidence: 0.7
provenance_state: extracted
---

# Logics-Parsing-V3：循环滑动窗口长文解析

阿里巴巴在 2026 云栖大会发布的开源文档解析模型，核心贡献是 **Structure-Aware Recurrent Parsing（循环滑动窗口推理框架）**：把长文档按顺序划分为连续页面窗口，每个窗口解析后提炼并更新一份"结构状态"（未闭合的标题路径、近期已闭合标题、跨页延续内容），传递给下一个窗口，最后统一重建为完整文档树^[raw/articles/logics-parsing-v3-长文跨页解析阿里云栖.md]。

## 核心机制

- **跨页结构状态传递**：模型翻页时延续前文结构状态，解决长文档翻页后的三类"失联"——跨页表格截断、标题层级错位、图文引用关系断裂^[raw/articles/logics-parsing-v3-长文跨页解析阿里云栖.md]。
- **超轻量**：模型仅 0.8B，在 MPDocBench-Parse 综合评测以 85.26 排名第一，领先第二名 PaddleOCR-VL-1.5 达 4.46 分；截断文本编辑距离降至 0.07，Heading TEDS 62.33（领先第二名 8.97 分）^[raw/articles/logics-parsing-v3-长文跨页解析阿里云栖.md]。
- **复杂内容还原**：密集文字、复杂表格、科学公式、化学符号，支持乐谱、思维导图、代码/伪代码还原^[raw/articles/logics-parsing-v3-长文跨页解析阿里云栖.md]。
- **开源**：GitHub `alibaba/Logics-Parsing`，HuggingFace `Logics-MLLM/Logics-Parsing-V3`，ModelScope Demo^[raw/articles/logics-parsing-v3-长文跨页解析阿里云栖.md]。

## 与 V2 的演进

V2（2026-03 发布）聚焦单页复杂版面解析与精准元素识别；V3 的关键跃迁是把"局部正确、全局失联"的问题显式建模为跨页状态传递——每页识别正确但整份文档没有真正读通，是长文档解析的核心痛点。前作 entity `entities/别让格式杀死思想logics-parsing-v2定义文档解析新边界.md` 已于 2026-09-07 质量闭环中归档。

## 可迁移价值

循环滑窗 + 结构状态传递的框架不限于 OCR/文档解析：任何"分块处理但需全局一致性"的多模态 pipeline（长视频理解、多轮 Agent 会话状态管理）都可借鉴"窗口输出 → 状态提炼 → 下一窗口注入"的模式。

## 相关

- [[entities/别让格式杀死思想logics-parsing-v2定义文档解析新边界]]（V2 前作，已归档）
- [[raw/articles/logics-parsing-v3-长文跨页解析阿里云栖|原文存档]]
