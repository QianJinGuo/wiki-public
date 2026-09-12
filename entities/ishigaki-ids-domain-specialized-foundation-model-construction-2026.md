---
title: Ishigaki-IDS — 建筑 BIM 领域专用基础模型（合成数据 + CPT/SFT/RLVR）
created: 2026-08-12
updated: 2026-09-12
type: entity
tags: [llm, training, rlvr, synthetic-data, domain-adaptation, foundation-model, aws, construction]
sources: [raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Ishigaki-IDS — 建筑 BIM 领域专用基础模型

→ [[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic|原文存档]] ^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md]

## 概述

Ishigaki-IDS 是 ONESTRUCTION（日本建筑科技创业公司）在 AWS GenAIIC 技术顾问下构建的建筑 BIM 领域专用基础模型，面向 IDS（Information Delivery Specifications，XML 标准）文件编写。建筑行业面临劳动力短缺，BIM 采用需要专业知识，IDS 文件编写需要掌握语法 + IFC 规则，通用模型难以准确产出其结构。Ishigaki-IDS 让非 BIM 专家也能审查和管理属性信息。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md]

## 三阶段训练管线

Ishigaki-IDS 基于 Qwen3（8B/14B/32B）开源模型，采用三阶段训练管线解决领域适配三挑战（数据稀缺、IFC 词汇注入、IDS 语法）。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md]

- **CPT（continued pre-training）**：注入 IDS/IFC 领域知识，语料 = web corpora + 领域专家参与的合成数据（合成数据覆盖大部分训练语料）
- **SFT（supervised fine-tuning）**：IDS 编写指令（CSV/自然语言）→ 期望 IDS 输出配对训练；但 SFT 遗留问题：看似合理但错误的 XML tag 选择、属性值错误
- **RLVR（reinforcement learning with verifiable rewards）**：用 buildingSMART 的 IDS-Audit-Tool 作奖励函数（检查 XML well-formedness、IDS 结构有效性、语义一致性），模型基于机械正确性信号迭代 —— 数据稀缺领域 RLVR 尤其合适，无需大量监督数据

## 关键方法

- **合成数据质量 > 数量**：领域专家参与合成数据创建是性能差异关键，Volume 本身无法达到同样效果
- **可验证奖励加速迭代**：机械验证器作自动奖励信号，数据稀缺场景下比人工评估更快
- **YaRN 上下文扩展**：确认 120k tokens 输入输出正确生成
- **评测**：自建 IDS-Bench（IFC 版本/建筑学科/日英双语/Implement-Structure-Content 轴）；Ishigaki-IDS 在 XML 结构合规与 IDS 结构合规接近 100%，IDS 内容一致性 >80%；通用 frontier 模型 XML well-formed 但 IDS 结构合规 <25%、内容一致性接近 0

## 基础设施

Amazon EC2 P5en（2 × p5en.48xlarge，NVIDIA H200）+ AWS ParallelCluster + Amazon FSx for Lustre，提供稳定多节点分布式训练与高吞吐数据访问。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md]

## 经验教训

- 合成数据质量 > 数量
- 可验证奖励加速迭代
- 稳定基础设施让实验自由（不调试集群）

## 深度分析

### 为什么通用 frontier 模型在结构化领域语言上系统性失败

通用 frontier 模型在 IDS 上的失败不是"知识不够"，而是三重约束无法同时满足：XML 语法本身通用且易学，但 IDS 的 tag 结构随"要交付或校验什么信息"而变化，IFC 词汇表是数千个术语的封闭映射（"beam" → `IfcBeam`），内容一致性还要求属性值语义正确。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 实测印证了这一点：通用模型能产出 well-formed 的 XML，却在 IDS 结构合规上低于约 25%、内容一致性接近 0%。三层约束是耦合的：tag 选错则结构不合规，IFC 映射错则内容一致性崩，联合正确率近似三层概率之积。更根本的是目标函数错配——预训练奖励"看起来合理"，IDS 却要求"机械上有效"，SFT 后遗留的典型缺陷正是看似合理但错误的 tag 选择与属性值。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 这类问题本质上更接近 [[concepts/llm-pretraining-vs-sft|预训练与 SFT 的分工]]，而非单纯的 prompt 工程问题。

### 合成数据里"专家参与"为何是质量分水岭而非数量

IDS 2024 年才发布，建筑领域公开网页内容本就少，数据稀缺的瓶颈不是 token 总量，而是决策边界的覆盖：某个交付场景该用哪个 tag、哪个术语映射到哪个 IFC 类，这些判断只存在于专家脑中，公开语料不会写。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 因此让模型自造 IDS 数据是危险的：自造数据会继承模型自身的错误先验，把表面合理但错误的结构放大成训练分布。ONESTRUCTION 的做法是让内部 IDS 专家参与合成数据创建，并从多个角度解释同一批 IDS 文档，使合成数据覆盖大部分训练语料，其结论明确——专家参与度而非数据量才是性能差异的来源。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 这与通用数据质量方法论一致：低资源域里数据的"分布正确性"比"体量"更接近瓶颈，可参看 [[concepts/data-quality-framework|数据质量框架]]。

### RLVR 用机械验证器做奖励在数据稀缺领域的杠杆

RLVR 用 buildingSMART 的 IDS-Audit-Tool 作奖励函数，它检查 XML well-formedness、IDS 结构有效性与语义一致性，等于把确定性审计工具变成近乎无限的奖励信号源。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 在数据稀缺领域这一点尤其关键：你没有足够标注数据做偏好对齐，却有一个可零成本重复调用的机械判定器。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 代价是奖励的地板由验证器决定——模型只会优化验证器能测的维度，测不到的"实际好不好用"不会自动变好，这是 [[concepts/verifier-paradox|验证器悖论]]的典型表现；最终仍靠 buildingSMART 联合 POC 让专家与非专家实测，才确认其实践价值。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 方法脉络见 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 概念]]。

### 领域专用小模型 vs 通用大模型的成本-合规取舍

选择 Qwen3 的 8B/14B/32B 尺寸谱系，让团队先在小规模做实验、再决定是否全量训练 32B，这是通用 API 模型给不了的实验自由度。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 专用模型的取舍集中在三点：一是成本与部署，自托管无 per-token 成本；二是数据合规，BIM 项目数据多受客户保密约束，本地可运行的小模型让敏感图纸不必出内网；三是场景适配度，配合 YaRN 把上下文扩到约 120k tokens，可把整份 IDS 与 IFC 文档一次性送入。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 代价是能力面收窄——专业训练可能挤压通用对话能力，所以 IDS-Bench 专门保留了通用对话能力这一监测轴。^[raw/articles/how-onestruction-built-the-ishigaki-ids-foundation-model-with-aws-genaiic.md] 同一逻辑在别的数据稀缺且合规敏感的行业同样成立，例如面向防御性网络安全的 [[entities/cybersecqwen-4b-why-defensive-cyber-needs-small-specialized-locally-runnable-mod|CyberSecQwen 4B]]（成本面可参看 [[concepts/context-window-economics|上下文窗口经济学]]）。

## 实践启示

对同样要在数据稀缺领域构建专用模型的团队，Ishigaki-IDS 的路径可以拆成五条可操作经验。

1. **领域验证器先行**：在收集数据之前先找到或形式化那个"机器可判定"的验证器（本例中是一个已存在的标准组织审计工具）。它一次性定义了三样东西——数据规格、评测标准、RL 阶段的奖励函数；没有它，"输出正确"就只是主观判断。
2. **把审计/合规工具变成奖励函数**：现有审计工具是免费且可无限调用的 reward signal，但它的覆盖范围就是模型质量的天花板。用之前先自问验证器测了哪些轴，测不到的轴必须另设专家评审兜底，否则模型会精准"应试"而非变有用。
3. **自建领域 benchmark 并分轴报告**：IDS-Bench 按 IFC 版本 × 建筑学科（建筑/结构/MEP/通用）× 语言（日/英）× Implement/Structure/Content 分轴评分。单一总分会把"哪一轴崩了"藏起来；分轴报告才能判断 RLVR 在哪一层起了作用、下一轮该改什么。
4. **YaRN 长上下文扩展要知道适用边界**：YaRN 能在不显著损失性能的前提下把窗口扩到约 120k tokens，项目验证了该长度下的生成正确性。但"窗口变长"不等于"尾部注意力可靠"——落地时应在真实的输入长度上验证检索与引用质量，别默认扩展即等价能力，并算清 token 增长带来的延迟与成本。
5. **合成数据必须专家在环**：把专家时间当作一等成本预算项，而不是收尾工作。批量自造数据的边际收益会迅速衰减，专家参与才是分水岭；在"多角度解释同一份源文档"这类能增加信息冗余的合成方式上投入，比单纯堆量更有效。

## 相关

- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 概念]] / [[entities/self-taught-rlvr|Self-taught RLVR]] / [[entities/overcoming-reward-signal-challenges-verifiable-rewards-based-reinforcement-learn|可验证奖励 RL]] / 合成数据
