---
title: "Model Genome: LLM 血统指纹识别方法论"
created: 2026-08-10
updated: 2026-10-25
type: entity
tags: [llm, model-lineage, fingerprinting, tokenizer, cka, open-weights, verification]
sources: [raw/articles/model-genome-llm-lineage-fingerprinting-2026]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Model Genome: LLM 血统指纹识别方法论

> 原文存档：[[raw/articles/model-genome-llm-lineage-fingerprinting-2026|原文存档]]

Model Genome 是 Hugging Face 社区文章提出的可复现管线：**当实验室宣称"自研、从零训练"基础模型时，外部人如何仅用公开产物验证这一说法**。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

## 三轴指纹识别

管线在三个轴上对模型做指纹识别，并合成为单一的 "genotype" 标签：^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

| 轴 | 方法 | 判别力 |
|----|------|--------|
| **架构**（config.json） | shape tuple (hidden_size, intermediate_size, num_hidden_layers, heads, kv) 与已知开源模型精确比对 | 五字段同时匹配 = 强证据 |
| **Tokenizer**（tokenizer.json） | 词汇表 min-overlap 比率 `\|A∩B\| / min(\|A\|,\|B\|)` | ~0.38 = "foreign brain, own language"（借架构训新 tokenizer）；1.000 = 直连微调 |
| **权重**（embedding CKA） | Linear CKA（旋转/各向同性尺度不变） | 可靠确认 from-scratch（近零），但无法强检测 derivation |

## 两个关键陷阱（方法论贡献）

1. **逐行余弦无用（旋转不变性）**：Transformer 隐藏空间无特权基，两个模型可在任意正交旋转下编码相同信息。逐行 cosine 把旋转视为不相似——对已知 from-scratch 模型和已知 Llama 衍生模型都测得近零均值 cosine，无法区分血统^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]
2. **CKA 帮助但不足**：from-scratch 模型对候选基座 CKA 近零（独立预训练干净证据）；但 continued-pretraining 衍生模型仅 ≈0.25，勉强高于同族不相关模型基线 ≈0.21。大规模训练重塑 embedding 使 CKA 在"衍生"侧失去判别力^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

**诚实结论**：权重轴可靠确认 from-scratch，但不是 derivation 的强检测器——config + tokenizer 指纹才是主要证据。

## Genotype 标签体系

| Genotype | 架构 | 权重 |
|----------|------|------|
| 🟢 Native | 自有 | from-scratch |
| 🔵 Adapted | 基本自有 | 借了一轴 |
| 🟡 Mixed | 部分 | 部分继承 |
| 🔴 Ported | 外来（精确匹配） | 继承 |

## 附加轴：注意力多样性作为原创性代理

config.json 中不同注意力机制的计数（layer_types, mamba2_d_state, hyena_filter_order, mla_kv_lora_rank 等）是架构原创性的廉价代理——最复杂的模型在一个栈中组合 mamba2、hyena、MLA、linear attention、gated-delta-net、native-sparse-attention 和 sliding-window。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

## 实证结果

应用于九家韩国组织的公开基础模型（LG K-EXAONE 2.0 750B 等）：部分精确匹配外来架构 + tokenizer（Ported）；部分自有架构和权重无外来匹配（Native）；多数介于两者之间。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

## 可复现性

三个函数即完整方法：`arch_fingerprint(repo)` / `tok_overlap(a, b)` / `linear_cka(X, Y)`，指向 Hub 上任意两个 repo 即可运行。Live demo + 完整数据集：Model Genome Korea Space。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

## 深度分析

### 一、为什么"血统"必须多轴交叉验证

每一轴都有廉价规避动作：config.json 可以复制后改名，tokenizer 可以整体替换，CKA 则要求双方共享词表与可比的 probe 数据。三个后门开在不同位置，互补才成立——架构轴抓"复制后改名"，tokenizer 轴抓"重构后仍继承词表"（~0.38 即"外国大脑、本地语言"），权重轴抓前两者都改过、却仍在权重层面同源的情形。单字段巧合毫无意义，五个字段同时命中才是指纹；而三轴同时失效只有一条路径——真的从头重训，那恰好是标签体系必须能命名的 Native 档，否则诚实的自研者会被判成未分类。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

### 二、两个陷阱的信号学意义

把陷阱当统计学问题读，比当技术细节更有用。逐行 cosine 无用（旋转不变性）是纯假阴性发生器：隐藏空间无特权基，真实衍生被判成不相似，from-scratch 与 Llama 衍生模型的均值 cosine 都近零。CKA 的陷阱则是双向的：continued-pretraining 衍生模型仅 ≈0.25，同族不相关模型的基线已 ≈0.21，差距薄到判不出（假阴性）；反之，共享上游预训练或同一语料谱系又会给出偏高的 CKA（假阳性）。架构轴同样假阳性：3584·18944·28·28·4 这类 7B 配置已是社区惯例，独立团队收敛到同一组数字并不稀奇。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md] 于是一条结论必须写进流程：指纹给出的是血统的**概率性证据而非证明**，它属于证据权衡（evidence-weighing）工作流，不属于指控（accusation）工作流——原文自己划线："工具报告血统，不报告过错。"

### 三、从指纹到治理

四档标签（Native/Adapted/Mixed/Ported）把连续相似度压成可操作的离散判定，让三类治理动作可执行：许可证与署名执行（衍生须守基座条款）、检测静默蒸馏（以"自研"名义复用权重，是最难取证的不当竞争）、可复现性审计（对"从零训练"声明做外部核查）。成本与激励错配：测量侧极便宜——三个函数、任意两个 Hub repo、无需训练与内部访问；执行侧不对称，有动机跑的是被侵权方与第三方审计者，被审方不会主动跑，结果一旦公开即声誉事件。"同一把尺子量每个模型、无例外"既是方法论要求，也是政治性设计。参见 [[concepts/model-distillation-compression]]。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

### 四、CKA 作为原创性代理的边界

linear CKA 的数值不是模型固有属性，而是"层对 + probe 集"的函数：比哪两层、用什么文本探测，结果都会移动。更根本的是，表示相似可源于数据与任务相似而非权重派生——两个都吃网页语料、tokenizer 相近的模型，中间层天然接近。故 CKA 高只是支持性证据，近零才是强反向证据，与原文"权重轴可靠确认 from-scratch、但不是 derivation 的强检测器"一致。注意力多样性轴是对此边界的补丁：若"同数据"足以解释相似，config 中的机制组合（mamba2、hyena、MLA 等，见 [[concepts/attention-mechanism|注意力机制]]）可区分"同数据、不同设计"与"同设计、同权重"；但计数只是廉价代理——模块堆得多不等于设计原创，把已有组件拼进一个栈本身就是拼装。^[raw/articles/model-genome-llm-lineage-fingerprinting-2026.md]

## 实践启示

1. **三轴一起报，不报单一相似度**：每个分数都附上 probe 条件（层对选择、词表交集规模、config 字段），否则同一个数字可被解读成完全相反的结论。
2. **锚定到具体版本**：记录 repo 的 commit、tokenizer 的哈希与 config 快照。模型会被重新上传，指纹没有版本锚点就无法复核。
3. **超阈值只触发人审，不触发判决**：高相似度是"请解释"的信号，不是"已定罪"的结论；先给对方留出用共享语料、共享上游等替代解释回应的机会。
4. **标签体系要版本化**：判定规则会随 probe 方法与阈值调整而变，历史判决必须能对应到当时所用的规则版本，否则旧结论不可解释。
5. **发布前先自证**：真正 from-scratch 的模型应在发布前对最可能的"祖先"跑一遍三轴，把近零 CKA 与不匹配的 config/tokenizer 一并公开，用先手证据堵住后续质疑。

## 相关实体

- [[entities/build-llm-from-scratch-7-chapters-zion|从零构建 LLM]] — 从零训练的工程路径（对应指纹的 from-scratch 侧）
- [[concepts/inference-optimization|推理优化]] — tokenizer/架构层优化背景
- [[entities/apple-silicon-costs-more-than-openrouter|本地推理成本分析]] — 模型实现差异的实证视角
