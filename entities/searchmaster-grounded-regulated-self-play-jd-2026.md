---
title: "SearchMaster：接地的受调节自博弈搜索 Agent 训练"
created: 2026-08-07
updated: 2026-08-07
type: entity
tags: [self-play, search-agent, agent-training, grpo, evidence-chain, reward-design, jd]
sources: [raw/articles/searchmaster-grounded-regulated-self-play-jd-2026]
confidence: 0.85
related:
  - entities/qwen-skill-self-play-hyman-2026
  - entities/llm-self-improvement-system-survey-zesearch-nlp-2026
  - entities/native-parallel-reasoner-icml2026
---

# SearchMaster：接地的受调节自博弈搜索 Agent 训练

## 摘要

京东未来研究院（JD Future Academy）SearchMaster（arXiv:2608.01822）：把搜索 Agent 自博弈训练形式化为「grounding（任务接地）+ regulating（行为调节）」问题。同一策略 πθ 兼任 Proposer（在本地搜索环境出多跳任务）与 Solver（用浏览器工具作答），冻结 Verifier 验证，GRPO 联合优化。针对自博弈三大误导信号给出三个控制：**ECG**（证据链生成器）抑制伪多跳题、**SDR**（搜索深度奖励）按最浅成功搜索深度定难度而非成功率、**OOP**（过度打开惩罚）按 open/search 比率正则工具使用。Qwen3.5-9B 六基准平均 38.19%→51.52%（+13.3），BrowseComp-Plus 30.12%→60.24%（+30.1），纯离线训练却在线五基准全部提升。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

## 自博弈的三大误导信号

现有搜索自博弈（Search Self-Play 预定义答案集、Dr. Zero 成功率难度）产出看似有用实则误导的训练信号：^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

1. **Pseudo Multi-Hop**：Proposer 浏览多文档却写出单文档可答的问题——表面多跳、实际浅查。naive 基线 47.1% 的任务是伪多跳
2. **成功率难度失真**：成功率无法区分「浅查即答」与「多步搜索才能答」——Solver 变强后任务自然变浅，hmin 衰减到 2-3
3. **Over-Opening 漂移**：训练中打开文档带来即时奖励，策略逐渐过度依赖 open——56.3% 的额外 open 只是重开已见文档，产生长而浅的轨迹

## 三控制机制（核心贡献）

### ECG：证据链接地的任务生成

Proposer 从种子文档出发，反复沿实体/事实链接跨文档延伸链条（选实体→搜关联文档→识别新实体→扩展），最后从完整链条出题，要求 full-chain necessity、无单文档可答。形式上每条 rollout 维护显式证据链 c_j = e_1 → e_2 → ··· → e_n。四道质量过滤器（Tool-use/Format/Validity/Parametric-knowledge）进一步移除无效候选。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

### SDR：搜索深度即难度

任务难度 = 成功 rollout 的**最浅**搜索深度 hmin = min_{i:z_i=1} h_i（Eq 3）——任何成功解能浅搜解决即视为 shortcut，不给高分。奖励 rsdr = r0 + (1−r0)·min(hmin/Hsdr, 1)（Eq 4），Hsdr=10 饱和防无限深搜奖励。对照成功率信号（1−|2C/K_S−1| 峰值在 50% 成功率）：SDR 让保留任务的 hmin 稳定在 8-10，成功率信号下随 Solver 变强衰减到 2-3。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

### OOP：比率惩罚而非次数惩罚

ρoop = clip((n_open/n_search − α_oop)/(β_oop − α_oop), 0, 1)（Eq 7，α=1.5 起罚/β=2.5 满罚），最终奖励 r = b − λ_oop·ρ_oop（Eq 8-9，λ=0.5）。用比率不用绝对次数：压制冗余打开（56.3% 重开已见文档）同时保留合法多文档探索。Proposer 与 Solver 共享策略，一次惩罚双角色都受益。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

## 训练设计

- **共享策略双角色 + 冻结 Verifier**：Verifier 是初始模型冻结副本；种子级 gate（K_P 任务最高奖励 ≤ r0=0.2 整种子丢弃）；只保留最高奖励通过任务的 K_S 条 Solver rollout 防样本主导（每种子贡献 K_P+K_S=16 样本）
- **GRPO**（Eq 10）：token 级 clipped objective + KL 正则，工具观测保留上下文但 mask 出 loss，梯度只流经模型生成 token
- **配置**：Qwen3.5-9B，lr 1e-6，clip (0.2, 0.28)，βKL=0.001；OpenResearcher 离线语料（约 15M 文档/11B token）+ Qwen3-Embedding-8B FAISS；20 迭代 × 64 种子 = 1280 种子；256K 上下文；上限 200 工具调用/rollout ^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

## 实验结果

**主结果**：Qwen3.5-9B 六基准平均 38.19%→51.52%（+13.3）；BrowseComp-Plus 30.12→60.24（+30.1）——9B 模型超过 gpt-oss-120B-high (42.89)、GPT-4.1 (35.42)、Claude Opus 4 (36.14)，逼近 o3 (63.49)。纯离线训练在全部 5 个在线基准提升（+6.8 GAIA 到 +18.2 WebWalkerQA），搜索行为泛化到开放网络。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

**消融**（BrowseComp-Plus）：naive self-play 45.18 → +ECG 53.25 → +SDR 52.89 → +ECG+SDR 57.71 → 全量 60.24——三机制单调互补。**任务质量**（GLM-5 判定）：True Multi-Hop 24.2%→78.6%，Invalid 28.7%→6.4%。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

## 与 Skill-SP/SESA 的家族关系

SearchMaster 与 Skill-SP（qwen-skill-self-play-hyman-2026 实体）同属 self-play 训练家族，但机制维度根本不同，形成互补：^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

| 维度 | Skill-SP / SESA | SearchMaster |
|------|----------------|-------------|
| 记忆机制 | Skill Card → Skill Bank 技能库进化（失败轨迹提炼可复用技能） | 无技能库——ECG/SDR/OOP 三控制在循环内做信号接地与调节 |
| 任务生成 | Proposer + 动态 skill 控制器 | Proposer + 显式证据链（ECG） |
| 难度信号 | gated curriculum（50% 正确率瞄准学习前沿） | 搜索深度 hmin（最浅成功解） |
| 工具调节 | 无显式工具使用惩罚 | OOP open/search 比率惩罚 |
| 验证 | 机器可读契约（单元测试/参考答案） | 冻结 Verifier + 四道过滤器 |

两篇都证明「自博弈数据无需人工标注 QA 对」：Skill-SP 靠 skill 库记忆复用扩能力，SearchMaster 靠接地+调节稳定信号。若飞关于「一次成功只是观察、要经归因回归才能变默认行为」的晋升边界（见 tencentdb-agent-memory-hierarchical 治理框架）在此有工程化体现——SDR 的 hmin 与 OOP 的比率惩罚本质上是给「哪种成功值得学」设置验证门槛。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

## 局限

训练仍需可搜索环境、每任务多条 Solver rollout、Verifier 调用（grounding/parametric filtering/correctness）——减少标注依赖但未消除全部成本；本地搜索环境限制开放网络全多样性覆盖；代码将在未来开源。^[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026.md]

## 相关实体

- [[entities/qwen-skill-self-play-hyman-2026|阿里 Qwen Skill-SP 自博弈]]
- [[entities/llm-self-improvement-system-survey-zesearch-nlp-2026|LLM 自改进系统综述]]
- [[entities/native-parallel-reasoner-icml2026|Native Parallel Reasoner]]

## 来源

→ [[raw/articles/searchmaster-grounded-regulated-self-play-jd-2026|论文原文存档]]
