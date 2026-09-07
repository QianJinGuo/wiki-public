---
title: "Om AI VLX-Seek: 3B 细粒度感知 VLM 架构"
created: 2026-07-03
updated: 2026-09-07
type: entity
tags: [vlm, multimodal, vision, detection, om-ai, region-token, fine-grained-perception, model-architecture]
sources: [raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Om AI VLX-Seek: 3B 细粒度感知 VLM 架构

VLX-Seek 是 Om AI（联汇）VLX 端侧流式多模态模型系列的第二弹，定位为**精准定位层**，解决 VLM "看得懂却看不准"的问题——即模型能回答"画面里有几个人"但无法稳定完成复杂指代和精确定位。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

## 核心洞察：坐标生成不是定位的最优解

传统 VLM 的定位方式依赖语言模型直接生成坐标（如 `[x1,y1,x2,y2]`），但**坐标不是自然语言**——LLM 擅长生成词语和短语，不擅长稳定输出精确数值序列。多目标场景下坐标序列会指数级拉长解码路径，且容易出现格式错误和幻觉检测。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

VLX-Seek 转换思路：**将定位任务从"坐标生成"转为"区域检索与引用"**——先把画面的物理实体转为可被语言模型读取、引用和推理的 region token，再让模型在候选区域之间比较、选择和指代。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

## 架构设计

### 区域 Token 机制

除全图视觉 token 和文本 token 外，模型接收一组**可寻址的区域 token**，每个对应图像中的一个候选区域并带区显编号（`<region0>`, `<region1>`...）。用户查询时，模型判断哪个区域匹配描述并输出区域索引——从"生成长数字序列"变成了"在候选视觉区域中做语言条件检索"。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

### OPN（Object Proposal Network）

OPN 先召回可能包含前景目标的候选区域，不判断类别，只提供视觉候选。它与 VLM 主体解耦——候选区域可来自 OPN、其他检测器、用户框选区域或 visual prompt。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

### HFRE（Hybrid Fine-grained Region Encoder）

HFRE 是 VLX-Seek 的**混合细粒度区域编码器**，使用双视觉路径架构：
- **主视觉编码器**：保留原始 VLM 语义对齐能力（"知道一个区域像什么"）
- **辅助视觉编码器**：提供更高分辨率的局部细节（"看到边界、纹理、小目标"）

通过 SimpleFP 模块为 ViT 特征补足多尺度表达，适应不同大小的候选区域。最后区域-语言连接器把区域特征投影到 LLM 嵌入空间。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

## 训练方法

两阶段训练策略：
1. **区域-语言对齐**：冻结主干 VLM，让新增 region token 进入 LLM 特征空间，压力集中在 HFRE、连接器和新增 token 上
2. **感知指令微调**：引入检测、指代理解、区域描述、计数、OCR 等多任务，同时混入常规 VLM 指令数据防止灾难性遗忘

关键设计约束：模型同时学会**拒识**——在目标不存在时回答"没有"，而非强行生成框。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

## Benchmark 表现

VLX-Seek-3B 在多项基准上超越更大参数量的模型：

| Benchmark | VLX-Seek-3B | 对比模型 | 
|-----------|-------------|----------|
| MSCOCO val2017 (mAP) | **45.3** | Gemini 3.1 Pro 41.4 |
| OVDEval 开放词汇检测 | **43.7** | - |
| ODinW13 开放词汇检测 | **48.4** | Qwen3.5-397B-A17B 47.0, Gemini 3 Pro 46.3 |
| RefCOCO/+/g Average | **88.7** | Qwen3-VL-8B 88.2, Gemini 3 Pro 84.1 |
| PixMo Count | **85.0** | Gemini 2.5 Pro 73.8, Qwen3-VL-8B 65.0 |

*数据来源：Om AI 官方发布* ^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

## 应用价值

对端侧设备来说，3B 规模和更低的解码开销意味着更低的部署门槛。具身系统要行动，必须先有稳定空间锚点——目标在哪、是哪一个、是否还在，直接影响跟随、避障、抓取和导航。VLX-Seek 的 region token 机制将检测与语言理解统一在同一框架下，为端侧具身视觉部署提供了可行路径。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md]

→ [[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

