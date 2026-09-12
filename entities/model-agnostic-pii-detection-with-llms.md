---
title: "Model-agnostic PII detection with LLMs（指令驱动、后端无关的 PII 检测器）"
created: 2026-09-11
updated: 2026-09-11
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

## 与本 wiki 的关联

- 与 [[entities/thehackernews-fake-openai-privacy-filter|OpenAI PrivacyFilter 讨论]] 直接对照：本文把 PrivacyFilter 作为基线之一（Core F1 80.7%，EC2 g4dn.xlarge，2.15s/条）。
- 与 [[entities/automatically-redact-pii-in-images-with-amazon-nova|图像 PII 脱敏]] 互补：那条走图像模态，本条走长文本 / 多语言自由文本。
- 训练语料治理侧面：[[entities/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026|OmniTable 数据策管]]。
- 安全与评测框架：[[concepts/ai-safety]]、[[concepts/evaluation-harness-design]]、[[concepts/agent-evaluation-benchmark-frameworks]]。
- Agent 场景的隐私外泄风险：[[entities/mosaicleaks-privacy-risks-deep-research-agents-servicenow|MosaicLeaks]]。

→ [[raw/articles/model-agnostic-pii-detection-with-llms|原文存档]]
