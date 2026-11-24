---
title: "Om AI VLX-Seek: 3B 细粒度感知 VLM 架构"
created: 2026-07-03
updated: 2026-11-10
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

## 深度分析

VLX-Seek 在 VLX 体系中承担精准定位层，处理的是"看得懂却看不准"的差距，也是继 [[entities/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|VLX-Flow]] 之后的系列第二层能力；它面向的[[concepts/embodied-intelligence-frontier|具身智能]]端侧场景中，定位结果要实时交给导航、抓取、避障或交互模块，推理效率本身就是核心指标。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:27-35,69]

### 坐标生成的脆弱性是结构性的

传统 VLM 让语言模型直接生成 `[x1,y1,x2,y2]`，但坐标不是自然语言——LLM 擅长产出词语和短语，并不擅长稳定输出精确数值。一个框四个坐标，顺序、归一化范围、标点格式、目标数量任何一处出错，结果就无法解析或明显偏离目标。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:59-63] 多目标进一步放大问题：十个目标就是几十个坐标 token，漏检和格式错误随序列变长而上升，而自回归解码每多一个 token 就多走一步，端侧场景的实时性首当其冲。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:65-69]

更深的错配在输出预算：坐标生成式模型把大量生成预算花在数值上，而非更快完成目标选择和空间决策；负样本下还会幻觉式检测，把不存在的目标定位到相似区域。这不是提示词工程能修复的缺陷，而是精确定位这一任务形式与语言模型生成方式的不天然匹配。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:71-77]

### 区域检索：把定位放回语言模型擅长的形态

VLX-Seek 的转换是先给候选、再做检索引用：画面物理实体先转成带显式编号的可寻址 region token（`<region0>`、`<region1>`…），查询到来时模型判断哪个区域最符合描述并输出区域索引——定位从"续写一串数值"变成"在候选视觉区域中做语言条件检索"，问题被放回语言模型本就擅长的比较、选择和指代。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:83-91]

输出侧用特殊 token 表达定位关系，如 `<ground>people</ground><object><region2><region10></object>` 表示文本实体 people 对应两个区域引用，解析比长坐标序列稳定；效率优势集中在多目标场景，目标越多坐标输出越长，而区域引用只输出索引，解码开销更低。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:133-143] 与把检测结果作为外部信息接入的检测增强 VLM 不同，VLX-Seek 让区域直接进入模型内部参与后续推理，检测、指代表达理解、区域描述、区域问答和计数由此落到同一套框架。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:145-151]

### HFRE 双路径：语义对齐与细粒度细节互补

单一视觉编码器难以两头兼顾：通用 VLM 的原始编码器已与语言模型对齐、擅长全局语义，但并非为精细检测训练，对小目标、边界、局部纹理和多尺度表达不足；检测类高分辨率编码器敏感于边缘和空间结构，却未必处在 LLM 能直接理解的语义空间。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:105-113] HFRE 以双视觉路径把两种能力接起来——主编码器保留语义对齐，辅助编码器提供高分辨率局部细节；SimpleFP 为 ViT 特征补足多尺度表达以适应大小悬殊的候选区域，区域-语言连接器再把区域特征投影到 LLM 嵌入空间，使 region token 同时携带高层语义与细粒度空间细节。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:115-129]

### 训练次序与拒识：叠加能力时的防退化设计

两阶段次序本身是设计：先做区域-语言对齐，冻结主干 VLM，把学习压力集中在 HFRE、连接器和新增特殊 token 上，先建立"一个 region token 对应一个视觉区域"的基础绑定；再进入感知指令微调，引入检测、指代理解、区域描述、区域推理、计数和 OCR，同时混入常规 VLM 指令数据保留问答与常识推理，防止检测能力挤压原有 VLM 能力。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:154-162] 拒识是这套框架的内生能力：模型同时学习什么时候不该找，目标不存在时回答"没有"而非强行指向某个区域——正对应坐标生成式模型在负样本上的幻觉式检测失败模式。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:164-166]

## 实践启示

1. 端侧具身视觉选型时，把"定位的输出形式"作为独立评估维度：多目标、高实时场景优先考虑区域检索式架构，其解码开销不随目标数量线性膨胀。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:143]
2. 构建 grounding 数据与评测时保留足量负样本，并把拒识作为显式验收项：对不存在目标回答"没有"的可靠性与框得准同等重要，直接决定幻觉检测率。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:73-75,164-166]
3. 任务形式的匹配比参数堆叠更能决定细粒度感知上限：VLX-Seek-3B 以 3B 参数在 MSCOCO val2017 达到 45.3 mAP（Gemini 3.1 Pro 为 41.4）、RefCOCO/+/g 平均 88.7、PixMo Count 85.0，反超更大模型。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:168-184]
4. 计数任务不要让模型凭画面印象估数：先检测并引用目标实例、再做聚合计数更可靠，区域引用机制可直接复用于该路径。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:184-186]
5. 双编码器是给既有 VLM 补细粒度能力的低侵入模式：不替换原始视觉编码器，以辅助编码器加连接器并行补足，训练时冻结主干、成本可控。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:109-117,156]
6. 候选区域来源是解耦的——OPN、其他检测器、用户框选区域或 visual prompt 均可，region token 机制可作为即插即用的定位层接入既有系统，而非整体推倒重建。^[raw/articles/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026.md:93-95]

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

