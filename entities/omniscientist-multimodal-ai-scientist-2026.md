---
title: "OmniScientist：全模态全学科 AI Scientist（直接感知原始证据）"
created: 2026-08-25
updated: 2026-08-25
type: entity
tags: [ai-scientist, omniscientist, multimodal, ai-for-science, scientific-discovery, research-agent, raw-data, evidence, perception, idea-check, rigour-check, claim-check, nus, oxford, first-party]
rating: v7c8
sources: [raw/articles/omniscientist-multimodal-ai-scientist-paper-2026]
confidence: 0.85
---

# OmniScientist：全模态全学科 AI Scientist（直接感知原始证据）

> 新加坡国立大学（NUS）与牛津大学团队（Bobo Li/Hao Fei 等）提出 OmniScientist，arXiv:2608.13558。核心主张：现有 AI-scientist 系统越来越 workflow-complete 却 evidence-incomplete——它们只在人类预处理后的文本/代码/标签/摘要上推理，丢失空间、时间、跨通道、过程性关系。OmniScientist 直接从异构原始证据做端到端跨学科研究，让感知贯穿完整科研生命周期。 ^[raw/articles/omniscientist-multimodal-ai-scientist-paper-2026.md]

## 4 类科学证据家族

按解释所需的主要推理方式把科学证据分成 4 个跨学科不变的家族（Table 2）：**Perceptual**（images/video/micrographs/radar/音频/3D）、**Symbolic**（文档/公式/序列/知识图谱/数学建模）、**Quantitative-statistical**（表/测量/分布/回归/显著性检验）、**Procedural/dynamic**（实验步骤/代码执行/agent traces/simulations/protocols）。关键洞见：产物都能序列化成 token 或通过代码访问，关键不是"什么能被序列化"而是"哪些关系能幸存于接口"——caption 丢局部空间结构、无序标量丢时间顺序。 ^[raw/articles/omniscientist-multimodal-ai-scientist-paper-2026.md]

## 框架架构：感知层 + 3 Agent + 确定性管线

一个 perception layer + 3 个自主 agent（Ideation/Experiment/Writeup）在确定性薄管线内运行，每阶段用 ReAct loop 交错观测、推理、行动，让证据塑造问题形成、实验设计、结果检查、科学论证。

- **感知层（4.1）**：先按推理范式把产物归到证据家族，家族内 modality 定义精确表示；优先做原生数值分析（FFT 峰值/趋势点直接从原始产物提取），仅空间/结构模式必要时才视觉渲染，视觉感知预算受限。任务上下文动态决定用数值特征、视觉表示或两者
- **Ideation（4.2）**：ReAct loop 建立 grounding → OpenAlex/Crossref 检索文献 → 至少 5 个候选 idea 评估 novelty/可行性 → 选最强定稿。Idea check 强制结构完整性、生成充分性、可执行性、泄漏/有效样本量检查，自动改写 overconfident 措辞（first→appears under-explored）
- **Experiment（4.3）**：受控 run_python 环境迭代代码生成 + 调试循环，感知层检查原始输入或自产图。设计含至少 4 个分析（主假设 + baselines/ablation/mechanism/sensitivity 对照）。Rigour check（Algorithm 1）验证真实执行、每数字可溯到真实输出、多重比较校正（含 demoted tests）、anti-HARKing（headline 必须来自受支撑分析）
- **Writeup（4.4）**：5 个结构规格固定 venue 风格（ML 带 Related Work/Limitations、生物医学末尾 Methods、化学合并 Results/Discussion），每节只从实验记录对应切片扩展，abstract 最后写保证数字一致；thesis planner 从受支撑分析选 headline；最终 meta-audit 检查 claim 对实验记录

三阶段检查（Idea/Rigour/Claim）都针对同一 **execution record**（source of truth）审计而非 agent 写的文本；实验崩溃或 null 结果返回 ideation。 ^[raw/articles/omniscientist-multimodal-ai-scientist-paper-2026.md]

## 评测：36 案例、5 学科、4 证据家族

- **任务设定**：单一 specification file 指定 dataset/subject/target property + 原始数据，系统产出证据接地论文，方法学交给 agent。演示套件 5 学科家族、36 案例、每案例一个真实公开数据集（12 个符号回归方程到 500 万边生物医学知识图谱）。扩展到新学科只需写 specification file，核心引擎零改动
- **模型**：reasoning backbone 驱动全部 3 阶段且是唯一被换组件；主用 Claude Sonnet 5，对比 GPT-5.6/GLM-5.2/Kimi K2.7/开源 Qwen3.5/Gemma-4。perception model 固定 Sonnet 5 不跟随 backbone（分数变化可归因于推理）。评分由 2 个家族外 judge（deepseek-v4-flash、gemini-2.5-flash-lite）
- **指标**：7 维 0-10（5 标准同行评审 + multimodal grounding/factual accuracy），composite=均值。分数与长度相关性极小（ρ=0.16）抗 verbosity 偏差。完整 3 阶段执行成本 $0.03–$4.34
- **主结果**：Sonnet 5 全部 36 案例完成、Overall 6.3（factual 7.7/soundness 7.0）；GLM-5.2 6.5、Kimi K2.7 6.2 接近；小模型明显下降。clarity 退化最小、factual/soundness 最贴 backbone 强度

## 核心贡献：直接感知原始证据

**盲眼消融**（Figure 7）：对比完整系统与只接收预计算标量特征、从不访问原始观测的盲眼变体——完整感知提升每个评测维度，最大增益在 multimodal grounding 和 scientific significance，其论文赢得 **85% 两两对比**。增益出现在科学实质：选择的问题、执行的分析、被证据支撑的结论。

**代表性证据接地发现**：地震学从 750 条噪声 STEAD traces 发现 21.7% 实际携带相干瞬态信号；病理学 held-out tile 分解为肿瘤/基质/淋巴混合物；海洋生物声学时间带宽积单独恢复 32 物种功能分组；符号回归 Cramér–Rao 形式匹配测量指数方差（8/8 定律）；无监督尺度不变描述符聚类出 4 个 morphotypes；并揭示评测陷阱（leave-one-family-out 误差比 k-fold 高 3.1×–7.0×）和元数据泄漏（记录协议 0.60→0.35 崩溃出源）。

**机制分析**：直接感知会影响问题选择、实验设计乃至最终研究路径，而不只是让论文写得更好——"看见"原始数据直接改变 AI 做什么研究。 ^[raw/articles/omniscientist-multimodal-ai-scientist-paper-2026.md]

## 关联实体

- [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AutoResearch L0-L4]] — 科研 agent 全流程自动化的同主题框架
- [[entities/polaris-zju-open-source-research-agent-2026|Polaris 开源科研 Agent]] — 开源科研 agent 体系
- [[entities/端到端论文生成系统假结论检出率92自动跑实验画图直出论文初稿|Spark-to-Paper 端到端论文生成]] — 同主题端到端论文生成，含假结论检出
- [[entities/sciagentgym-benchmark-multi-step-scientific-tool-use|SciAgentGym 科学工具基准]] — 科研 agent 评测基准
- [[entities/nature-ai-scientific-assistant-google-futurehouse|Google AI 科研助手]] — 科研助理体系
- [[raw/articles/omniscientist-multimodal-ai-scientist-paper-2026|原文存档（论文）]]
- [[raw/articles/omniscientist-multimodal-ai-scientist-oscholar-2026|原文存档（Oscholar 解读）]]
