---
title: "Model-agnostic PII detection with LLMs（指令驱动、后端无关的 PII 检测器）"
created: 2026-09-11
updated: 2026-11-15
type: entity
tags: [llm, pii, privacy, data-curation, evaluation, benchmark, bedrock]
sources: [raw/articles/model-agnostic-pii-detection-with-llms]
confidence: 0.8
---

# Model-agnostic PII detection with LLMs（指令驱动、后端无关的 PII 检测器）

AWS（Amazon AGI Foundations Responsible AI team）发布的 LLM 化 PII 检测方案：把「检测什么」变成 prompt 里的文本 schema，把「用哪个模型」变成可替换的后端接口，从而摆脱传统 token-classification 标注器「实体集在训练时冻结 + 绑定单一部署」的两大约束。正文给出 5 个公开语料、9 个 LLM 检测器的 span-level Core F1 横向对比，并附开源包 `pii-detector`。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 问题框架：为什么 PII 检测该由 LLM 重写

微调语料本身充满 PII（姓名、地址、邮箱电话、身份证件号、银行账户、生日），模型会在训练中记忆并在不该出现的 prompt 下复现。传统方案是双向 token-classification tagger：实体类型在训练时固定，遇到自定义语料引入的工号、加密钱包地址等就落在 schema 之外，加一类就要重新标注 + 重训，且绑定单一模型与单一部署。LLM 把这三个约束同时变成配置：指令在推理时读取，于是「检测哪些实体」「输出什么格式」「跑在哪个后端」都从代码变成 prompt 与接口参数。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 核心设计：两个杠杆

**杠杆一 — 模型后端可替换（Inferencer 接口）。** 检测逻辑全在 instruction 与一层薄解析里，模型通过统一接口 Inferencer 访问，语义是「messages in, text out」。仓库自带 Amazon Bedrock 适配器（Converse API 的薄封装），同一接口也可接自托管开源模型（如单卡 OSS-GPT 20B），覆盖无法访问托管 API 的隔离/气隙环境。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

**杠杆二 — 实体集即文本（Ext 配置）。** schema 存在单一 system prompt 模板里：15 个实体类别各带一行定义、一份 do-not-flag 列表、可选 few-shot 与输入文本。增删类别是改一行指令，无需重训、无需重新部署。模型只返回 JSON 列表（`pii_entity_type` + 原文 `pii_entity_value`），**不返回字符偏移**——LLM 无法可靠产出偏移，由后处理恢复。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 基准：5 语料 × 9 检测器，span-level Core F1

评测取 Hugging Face 上 5 个带 ground-truth span 的公开 PII 语料，每个采样约 1 万行，合计 **49,365 条记录 / 222,114 个 ground-truth core span / 8 种语言**（de/en/es/fr/hi/it/nl/te），领域从多语言合成档案到英文 HR 与客服文档。匹配规则是预测 span 与标注的 exact (start, end, label) 重叠（IoU = 1.0），报 Precision / Recall / F1。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

跨检测器比较的首要障碍是标签体系不一致（`PRIVATE_NAMES` vs `NAME`、`street_address` vs `street`），因此所有原始标签（检测器输出与数据集标注）统一映射到 12 类 canonical taxonomy（NAME / ADDRESS / CONTACT_INFO / DATE / AGE / SSN / FINANCIAL / IP_ADDRESS / URL / USERNAME / PASSWORD / ID_NUMBER），且每个检测器只在「它与数据集共同声明的标签范围交集」上计分——不为它从未声称支持的类别扣分。头条指标 Core F1 即这 12 类的分数，另有覆盖数据集特有类别的 extended-entity F1。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

| 检测器 | 后端 | Core F1 | 单条耗时 (s) |
|---|---|---|---|
| Mistral Large 3 | Bedrock | 83.1% | 1.16 |
| OSS-GPT 20B | EC2 (g5.12xlarge) | 81.6% | 1.17 |
| PrivacyFilter | EC2 (g4dn.xlarge) | 80.7% | 2.15 |
| OSS-GPT 120B | Bedrock | 79.4% | 3.91 |
| Qwen3.6-27B | EC2 (g5.12xlarge) | 79.5% | 12.79 |
| Nova Lite 2 | Bedrock | 74.9% | 0.77 |

两条可迁移结论：**延迟由模型（推理冗长度与架构）决定而非参数量**——同为 ~20B 量级，OSS-GPT 20B 约 1.2s，Qwen3.6-27B 约 12.8s；**后端可自由替换**——OSS-GPT 20B 在 EC2（81.6%）与 Bedrock（81.3%）上只差 0.3 个点。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 实体集杠杆：Ext 配置把罕见实体 F1 拉高约 6 倍

base 配置对数据集特有类别（occupation、company name、crypto-wallet 地址、vehicle identifier、user-agent）几乎无概念，分数接近 0；Ext 配置只做指令层改动——加入这些类别的定义与少量示例，并删除冲突的 do-not-flag 行（如 company name 成为目标后，把 business addresses 移出 public 列表）——无需新模型、无需重训，扩展实体 F1 从约 12% 跳到约 73%，同时 core F1 不变或略升。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

| 检测器 | 后端 | Ext-entity F1 (base ▸ Ext) | Core F1 (base ▸ Ext) |
|---|---|---|---|
| Qwen3.6-35B-A3B | EC2 | 9.4% ▸ 80.5% | 79.4% ▸ 83.5% |
| OSS-GPT 20B | EC2 | 12.1% ▸ 73.3% | 81.6% ▸ 83.1% |
| Mistral Large 3 | Bedrock | 17.3% ▸ 72.7% | 83.1% ▸ 89.1% |
| Gemma-4-E4B-it | EC2 | 12.5% ▸ 72.5% | 79.4% ▸ 83.8% |

在 ai4privacy_500k 分解上，OSS-GPT 20B 在全部 8 种语言保持 83–90% Core F1（含非拉丁的 Hindi / Telugu），在 SSN、financial、ID number 等高危标识上 **>95%**；共同短板是 DATE，约 50%——span 边界与格式本身存在歧义。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 工程要点：后处理三步与「幻觉标签回收」

模型返回的是纯文本值，落成可用的 span 列表要经三步（`pii_detector/detector.py`）：JSON 解析 → 偏移计算（用正则把值定位回源文本）→ **幻觉标签回收**：LLM 常输出近似标签（`DATE` 之于 `DATES`、`EMAIL` 之于 `CONTACT_INFO`），每个输出标签先按 prompt 自身词表经形态学与别名表重新归位，三级都映射不上的标 `UNK` 而非强行塞入，让真正的幻觉保持可见。这条「不把不确定输出强行归一」的工程选择值得迁移。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

工程交付：`pii-detector` 包 + 开源仓库 `aws-samples/sample-llm-pii-detection`，Python 3.11+，唯一运行时依赖 boto3；完整 prompt 在 `pii_detector/templates.py`，Bedrock 适配器在 `pii_detector/bedrock_inferencer.py`，完整评测表在 `docs/benchmarks.md`。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 边界与注意事项

- **标签归一化会掩盖绝对能力差异**：12 类 canonical taxonomy + 只在自己的标签交集上计分，使横向数字可比但非绝对——换一套映射，分数会移动。引用该表的数字时必须带上这一前提。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
- DATE 约 50% F1 是方案本身的已知弱项，不是实现缺陷；涉及日期类敏感信息时不要假设「LLM 检测器已够用」。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
- 单条耗时是「总墙钟时间反推回单条」的估算值（实际按多 worker 并行跑整个语料），只作量级参考。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
- 评测基础设施仍带 AWS 色彩（Bedrock Converse API 为默认路径），但**检测方法（prompt-as-schema + 可替换后端 + span 级 benchmark 协议）与后端无耦合**，换任何满足 messages-in/text-out 的推理端点都成立。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 深度分析

### 两个杠杆为何复合：能力解耦后的自由度乘积

传统 tagger 把「检测什么」（实体 schema）和「用什么算」（模型权重）焊死在同一份 checkpoint 里，两个维度只能通过重训联动。本方案把二者拆成互相正交的配置轴：实体集是 system prompt 里的文本（**Ext 配置**），模型是 messages-in/text-out 接口后的任意后端（Inferencer）。复合性来自「正交改变、各自生效」：Ext 式指令改动对所有受测模型同样生效——extended-entity F1 从约 12% 升到约 73%，无一例外；换后端则不改变 prompt 的语义，OSS-GPT 20B 在 EC2 与 Bedrock 上只差 0.3 个点。两个自由度相乘意味着 N 个模型 × M 个实体集配置都是合法组合，且任一轴上的改进（更强的模型、更全的 schema）立即被所有现有组合继承，而固定 schema 的 tagger 每次都要整体重训。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

### 基准方法论批判：可比性是买来的，代价是绝对意义

评测设计的核心困难是标签体系不可通约（`PRIVATE_NAMES` vs `NAME`、`street_address` vs `street`），解法是把检测器输出与数据集标注的原始标签统一映射到 12 类 canonical taxonomy，且每个检测器只在「它与数据集共同声明的标签范围交集」上计分。这买来了横向可比性，代价是分数变成「映射函数的函数」：换一套映射，数字会移动；交集计分还隐藏了绝对覆盖差异——声明范围窄的检测器天然回避了难题类别。其次，exact (start, end, label) 的 IoU = 1.0 匹配偏严，DATE 类约 50% 的低分有相当部分来自 span 边界与格式的真实歧义而非漏检。最后，单条耗时是「多 worker 并行总墙钟反推回单条」的估算值，只作量级参考。引用这份榜单的任何数字，都必须同时引用这三个前提，否则数字不可复现。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

### 幻觉标签回收：三级归位与 UNK 逃逸口

LLM 输出的标签常是「近似正确」的（`DATE` 之于 `DATES`、`EMAIL` 之于 `CONTACT_INFO`）。回收分三级：先按形态学变化归位，再查人工维护的别名表，都失败则标 `UNK`。关键工程决策在第三级——不强行归一。强行塞入最近的类别会把「模型真的产生了 schema 外标签」这一信号抹掉，下游从此无法区分「归位成功」与「幻觉被掩盖」；UNK 让失败保持可见，可统计、可审计，进而反哺 prompt 迭代。与之配套的是职责外移：模型只返回文本值，字符偏移由正则在确定性后处理中恢复，因为 LLM 无法可靠产出偏移——把模型做不稳的职责全部移出模型，是整个方案的隐性设计原则。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

### 成本/延迟画像：参数量不解释延迟

9 个检测器中最反直觉的一条：同为 ~20B 量级，OSS-GPT 20B 约 1.2s/条而 Qwen3.6-27B 约 12.8s，差一个数量级——延迟由推理冗长度（reasoning verbosity）与架构决定，而非参数量。选型因此是三维的（Core F1、单条耗时、token 成本）而非单一排行榜：Nova Lite 2 以 74.9% 换 0.77s/条，适合大批量粗筛；Mistral Large 3（83.1%、1.16s）适合质量敏感路径。Qwen3.6-27B 在这份表里几乎被 OSS-GPT 20B 处处支配——Core F1 更低（79.5% vs 81.6%）且慢约 10 倍——说明通用模型榜单的排名不等于检测场景的排名。当然，延迟数字本身带着前述「反推估算」的误差带，只该用来做量级判断。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 实践启示

1. **把实体集当配置管理，不当训练资产。** 新增工号、加密钱包地址等域内类别时，先试 Ext 式指令改动（加定义 + 少量 worked example + 删除冲突的 do-not-flag 行，如把 business addresses 移出 public 列表），重训是最后手段。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
2. **接口先行。** 任何 LLM 检测/抽取管线都应从 messages-in/text-out 的 Inferencer 式接口起步，让模型成为部署后期可换的参数，而不是在业务代码里散落 provider SDK 调用——这同时解决了气隙环境的接入问题。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
3. **不要让 LLM 返回字符偏移。** 让它回文本值，偏移在确定性后处理里恢复；同理，凡 LLM 做不稳的定位/数值职责都应外移到代码。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
4. **为标签归一化保留 UNK 逃逸口。** 形态学 → 别名表 → 放弃的三级映射，比「总归入最近类」更能暴露真幻觉；静默纠错在结构化输出解析和工具调用路由中同样是反模式。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
5. **引用跨检测器 benchmark 数字时附上计分前提**：canonical 映射、交集计分、IoU = 1.0、延迟为反推估算。这套基准与 [[concepts/evaluation-harness-design]] 讨论的可比性/公平性权衡同构，换一套映射或放宽 IoU，结论可能移动。^[raw/articles/model-agnostic-pii-detection-with-llms.md]
6. **选检测模型先测延迟与成本，再信参数量与榜单。** 小模型可能以 <1s/条 达到可用质量，慢模型在整批清洗场景可能整体不可用；与 [[entities/thehackernews-fake-openai-privacy-filter|PrivacyFilter 基线争议]] 合并阅读，能看清「榜单名次 ≠ 场景最优」。^[raw/articles/model-agnostic-pii-detection-with-llms.md]

## 与本 wiki 的关联

- 与 [[entities/thehackernews-fake-openai-privacy-filter|OpenAI PrivacyFilter 讨论]] 直接对照：本文把 PrivacyFilter 作为基线之一（Core F1 80.7%，EC2 g4dn.xlarge，2.15s/条）。
- 与 [[entities/automatically-redact-pii-in-images-with-amazon-nova|图像 PII 脱敏]] 互补：那条走图像模态，本条走长文本 / 多语言自由文本。
- 训练语料治理侧面：[[entities/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026|OmniTable 数据策管]]。
- 安全与评测框架：[[concepts/ai-safety]]、[[concepts/evaluation-harness-design]]、[[concepts/agent-evaluation-benchmark-frameworks]]。
- Agent 场景的隐私外泄风险：[[entities/mosaicleaks-privacy-risks-deep-research-agents-servicenow|MosaicLeaks]]。

→ [[raw/articles/model-agnostic-pii-detection-with-llms|原文存档]]
