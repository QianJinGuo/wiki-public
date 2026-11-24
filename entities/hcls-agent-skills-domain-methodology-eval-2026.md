---
title: "HCLS 领域技能包：38 个开源 Agent Skills 的方法论化与量化评估（AWS）"
created: 2026-09-17
updated: 2026-11-16
type: entity
tags: [agent-skills, harness, skill-engineering, progressive-disclosure, context-engineering, evaluation, llm-judge, win-rate, cohens-d, hcls, domain-specialization, multi-agent-routing]
sources: [raw/articles/hcls-agent-skills-domain-methodology-eval-2026]
confidence: 0.85
---

# HCLS 领域技能包：38 个开源 Agent Skills 的方法论化与量化评估（AWS）

AWS 的 HCLS（healthcare & life sciences）团队开源了 38 个领域 Agent Skills（11 个领域，MIT-0，遵循 agentskills.io 开放标准），并用 410 条领域 prompt 在两套 harness 上做了成对评估，量化了"给 Agent 装上领域决策程序"的收益。本文的价值不在技能本身，而在它给出的**技能收益测量方法论**（win rate + Cohen's d）与**上下文工程解法**（38 技能 80K tokens → 协调者/专家路由）。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 核心命题：模型缺的不是事实，是"决策程序"

基线 Agent 在 HCLS 场景会系统性误用决策框架：叫它按 ACMG/AMP 标准分类 TP53 错义变异，它会引对框架却错用证据类别、跳过人群频率阈值、甚至编造计算预测器分数。这不是知识缺口——模型在训练语料和 system prompt 里都见过指南——而是缺少执业者在多年训练中内化的**结构化推理程序**，导致变异解读、理赔裁定、临床试验设计、影像分析等环节出现"看起来正确但判据用错"的静默失败。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

技能与其他领域专业化路径的区别被明确划出三条界：RAG 从索引文档里检索有限段落来增强生成；技能编码的是**决策程序本身连同错误条件**；技能也不是微调，而是按查询触发模式选择性激活的结构化 prompt（SKILL.md）。技能同时具备三个工程属性——可审计（每条判据都是人可读的 markdown，不在权重里）、可移植（Amazon Bedrock AgentCore、Strands Agents SDK、Kiro、Quick Desktop、Claude Code、OpenAI Codex 等 20+ 宿主零改造复用）、可维护（年度医疗政策或新实验标准变更时改文本文件，不重训模型）。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 双分类：reasoning skills 与 pipeline skills

该集合按用途切成两类，这条分类法本身就是可迁移的技能设计经验。**Reasoning skills** 编码方法与决策框架，决定 Agent"怎么想"：例如 genomic-variant-interpretation 技能把完整 ACMG/AMP 分类框架（证据类别、人群频率阈值、计算预测器 cut-off）写进技能。**Pipeline skills** 编码工具特定命令、已验证参数与代码模板，产出可直接运行的工件：例如 variant-calling 给出 GATK4 HaplotypeCaller 的正确注释组、VQSR tranche 灵敏度目标与 Mutect2 tumor-normal 配置。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

双分类的实际含义是：推理技能靠决策树 + 编号过程发挥价值，管道技能靠参数表 + 版本特定 gotcha 发挥价值；设计指南 SKILL_DESIGN_GUIDE.md 记录的正是这类与高胜率相关的结构特征（决策树、阈值表、坑清单、响应格式段）。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 上下文工程：38 技能 ≈ 80K tokens → 协调者/专家路由

把 38 个技能全量塞进单个 Agent 上下文约 80K tokens。大上下文模型能承受，但会制造一个典型的上下文工程问题：每次查询都要从 38 个技能里选对子集，而无关技能内容会争夺注意力。另一条路是显式调用（`/risk-adjustment`），但要求提问者事先知道该调哪个技能——这恰好是技能本要弥补的专家经验缺口。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

Kiro CLI 的多智能体架构同时解决这两点：一个不加载任何技能的轻量协调者做意图分类，把查询路由到 8 个领域专家，每个专家只加载自己的相关技能（约 15K tokens/专家）；配置落在 JSON agent 文件里（coordinator 配置与其路由逻辑、specialist 配置与技能挂载方式均可查）。Strands SDK 侧则用 `AgentSkills(skills="./skills/")` 挂载技能、`MultiAgentOrchestrator` 做协调；AgentCore harness 还支持在环境级注入技能，使该 harness 下运行的所有 Agent 都能拿到，并附带托管、自动扩缩、安全边界与可观测能力。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 评估设计：410 prompt × 2 harness × 5 维度 × LLM judge

评估用成对比较（pairwise）衡量技能影响：410 条领域 prompt（380 条单技能 + 30 条跨技能），两种 harness 配置并行跑。配置一是 Kiro CLI（Auto 模型让 Kiro 自选最优模型，Agent 可访问 thinking 工具与文件读操作）；配置二是 Strands Agents SDK 搭的 Agent（模型显式钉死 Claude Sonnet 4.6，`callback_handler=None`），thinking 工具在两条件下对称提供。两配置内部各比较两组：无技能的基线 Agent，与通过渐进式加载拿到全部 38 个技能的有技能 Agent。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

判官用 Amazon Bedrock 上的 Claude Opus 4.7，按 5 个维度 0–100 打分：scientific accuracy（事实/机制/引用/领域知识正确性）、coherence（推理链与内部一致性）、relevance（覆盖 prompt 各部分且不跑题）、critical thinking（挑战假设、指出局限、给出替代方案，而非单一未批判答案）、actionability（可立即执行的具体步骤、参数、可运行命令）。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

指标选择是本文最可复用的一处工程决策：LLM judge 存在 **score compression**（分数聚簇，+1.5 这样的原始 delta 无法解释），因此不报原始分差，改报两个主指标——**win rate**（技能条件胜出的 prompt 百分比，直观且对尺度压缩鲁棒）与 **Cohen's d**（均值差除以 pooled 标准差，衡量提升相对自然方差的量级；0.2 小、0.5 中、0.8 大）。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 结果：胜率 69.5–85.9%，最弱基线收益最大

| 指标 | Kiro CLI | Strands Agent |
|---|---|---|
| 评估 prompt 数 | 410 | 410 |
| Skills 总体 WR (d) | 69.5% (0.39) | 85.9% (0.97) |
| Critical thinking WR (d) | 78.0% (0.65) | 85.1% (1.03) |
| Scientific accuracy WR (d) | 69.3% (0.34) | 86.2% (0.85) |
| Actionability WR (d) | 68.0% (0.37) | 77.3% (0.56) |
| 基线-收益相关 r | −0.59 | −0.61 |
| 最大方差收缩 | −61.9% | −52.1% |

技能在两套 harness 下分别赢下 69.5% 与 85.9% 的成对比较，且 critical thinking、actionability、scientific accuracy 三个维度同时改善。最强信号落在 critical thinking，印证技能的主要贡献是**方法学**的：教 Agent 该套用哪些框架、该挑战哪些假设、该标注哪些限制，而不是补充基础模型本来可能已有的事实。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

基线越弱、技能收益越大。基线质量与技能收益的 Pearson 相关为 −0.59（Kiro）与 −0.61（Strands）。按基线分数分档后：Kiro 弱档（<80）15 条从 75.8 提到 84.4（+8.7，WR 87%）、中档（80–90）227 条 +2.3（WR 79%）、强档（>90）168 条 −0.3（WR 55%）；Strands 弱档 46 条 77.0→84.9（+7.9，WR 96%）、中档 325 条 +3.7（WR 89%）、强档 39 条 ±0.0（WR 54%）。弱档集中在临床数据、医疗运营、基因组学，典型形态是需要多步监管程序的场景——模型是"近似"而非精确应用这些流程。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

强档并非铁板一块：跨领域推理技能在 90.2 的强基线上仍拿到 80% 胜率，说明只要技能教的是模型自己不会应用的决策程序，方法学框架在整个质量谱上都能增值。此外技能显著压缩输出方差——Kiro CLI 下 clinical-data 响应的判官标准差从 6.8 降到 3.3（−51%），意味着输出更一致地遵循技能编码的框架；在一致性权重不亚于平均分的受监管流程里，这是独立于均值提升的收益。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 可复现的评估框架与自定义路径

集合附一套可自跑的评估框架：`python eval/generate_prompts.py --count 30` 生成评估 prompt，`python -m eval.run --skills ./my-custom-skills/ --parallel 2` 跑成对评估，`eval/build_review.py` 产出 review.html 仪表盘（含评分、领域分解、prompt 与响应）；判官 prompt 与执行脚本均在仓库中（awslabs/hcls-agent-skills，示例镜像 aws-samples/sample-hcls-agent-skills）。定制路径由三份文档覆盖：CUSTOMIZING.md（改阈值、加组织内规则如 LCD 代码/处方分层治疗/内部流程）、SKILL_DESIGN_GUIDE.md（写作模式与结构特征）、QUALITY_CHECKLIST.md（frontmatter、结构、内容质量、测试的合并前清单）。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 边界与解读约束

三点限制决定了结论的可迁移范围：其一，判官是单一供应商模型（Claude Opus 4.7），胜率与效应量都建立在它的评分行为上；其二，收益幅度强依赖 harness——同一技能集在 Kiro CLI 上是 69.5% (d=0.39)，在 Strands 上是 85.9% (d=0.97)，即"技能收益依 harness 而定"本身就是必须记录的观测，不能只报一个数；其三，领域绑定 HCLS，向其他垂直领域迁移时需重新跑评估。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

## 深度分析

### 失败形态：引用即误用的静默失效

三条用例呈现同一种失效结构——模型"引对框架、用错判据"：RAF 管道漏掉 V28 层级消解（HCC 18 应压过 HCC 19）甚至先求和再消解，分数系统性虚高并埋下 RADV 审计责任；影像管道把偏置场校正排在颅骨剥离之后，脑边界残留的强度不均会侵蚀脑掩膜，在颞叶与额叶区域引入系统性偏倚；药物重定位回答罗列已知 TGFBR1 抑制剂，却没有证据层级、排序判据与可转化性评估。这类输出的共同点是"看起来合理"，错误藏在判据的应用顺序、阈值与版本细节里，靠通读很难发现——这正是技能把"决策程序连同错误条件"显式写成文本、而非依赖模型隐式近似的核心理由。

### 收益归因：方法学贡献大于事实贡献

最强信号落在 critical thinking（WR 78.0–85.1%，d 0.65–1.03）而非 scientific accuracy，说明技能的主要产出是"该套用哪个框架、该挑战哪些假设、该标注哪些限制"。弱档基线的收益集中在多步监管程序场景——模型只能"近似"流程而非精确执行——而强档里跨领域推理技能在 90.2 基线上仍有 80% 胜率。两个事实合起来给出技能增值的充分条件：它教一条模型不会主动执行的决策程序；收益与基线强弱并非单调反比，"强基线无收益"的结论只对事实补充型内容成立。

### 方差收缩：一致性是独立于均值的收益

clinical-data 响应的判官标准差从 6.8 收缩到 3.3（−51%），全库最大收缩 −52% 至 −62%。这组数字常被平均分掩盖，但它回答的是另一个问题：技能充当了行为约束，把输出分布拉向技能编码的框架，使每次回答都稳定落在同一流程内。在受监管流程中，"可复现、可审计的一致性"与"平均质量更高"是两类不同资产——前者直接决定评审与审计成本，且无法靠换更强模型自动获得。

### 路由经济学：意图分类与领域推理的分工

全量加载 80K tokens 的真实代价不是放不下，而是每次查询都要求模型在 38 项里选对子集，无关内容争夺注意力；显式调用则把选择负担推回用户，而"知道该调哪个技能"恰是技能要替代的专家知识。协调者/专家架构把矛盾拆开：不加载技能的轻量协调者专职意图分类，8 个专家各自挂载约 15K tokens 的领域技能。这本质上是把技能选择从"单次注意力竞争"转移给专门的路由组件，与 [[concepts/agent-orchestration-patterns|Agent 编排模式]] 中按职责分层的通用做法一致；配置落在可查的 JSON agent 文件里，也让路由逻辑本身可审计。

### 测量学：score compression 下的成对报告

判官分数聚簇使原始 delta（如 +1.5）不可解释，本文的替代方案可整体迁移：报对尺度压缩鲁棒的 win rate，配以用 pooled 标准差归一的 Cohen's d；再按基线分数分档，把"平均收益"拆成"对谁有收益"（r≈−0.6 的负相关）。同一技能集跨 harness 从 69.5% 摆到 85.9% 还揭示了第三条规则：效应量必须绑定 harness 配置报告，脱离配置的单点数字不可迁移——测量协议本身是结论的一部分。

## 实践启示

1. 先诊断失败类型再决定投入：若模型"引对框架但用错判据"（顺序、阈值、版本细节出错），写技能是正确工具；若缺的是事实或冷门资料，RAG 检索或微调更合适——技能对事实缺口收益有限。
2. 按双分类决定写法：reasoning skill 用决策树、编号流程、阈值表组织判据，并显式写出错误条件；pipeline skill 用参数表加版本特定坑清单，保证产出可直接运行。
3. 评估技能时用成对 win rate + Cohen's d，按基线分数分档呈现结果；不要报 LLM judge 的原始分差——分数聚簇会把它变成噪声。
4. 技能收益必须逐 harness 复测并在报告里标注 harness 配置；同一技能集在不同 harness 上的 WR 可相差 16 个百分点。
5. 在审计与合规敏感场景，把输出方差收缩作为一级指标与均值并列监测；技能带来的稳定性是独立收益，换更强模型替代不了。
6. 技能库超过十几个后不要全量加载：用不加载技能的轻量协调者做意图分类，按领域专家分片挂载技能（约 15K tokens/专家），路由配置放在可审计的 JSON agent 文件里；同时保留配套评估脚本（生成 prompt → 成对评估 → review 仪表盘）作为技能改动的回归门。

## 与既有 wiki 知识的接口

本文与技能评估轴上的既有页面同源可比：[[entities/agent-skill-writing-evaluation|技能写作评估]] 讨论技能的评估维度，[[entities/alibaba-skill-up-agent-skill-evaluation-framework-2026|阿里 Skill-Up 评估框架]] 给出企业侧评估框架，本文补齐的是"成对胜率 + 效应量 + 基线分档"这一套可直接复用的测量学。上下文经济性上，38 技能 80K tokens 与专家 15K tokens 的对比可挂到 [[concepts/context-window-economics|上下文窗口经济学]] 与 [[concepts/context-engineering|上下文工程]]。技能结构层面，SKILL.md + YAML frontmatter + 渐进式披露与 [[concepts/hermes-agent-skill|Agent Skill 概念]]、[[entities/agent-skills-development-guide|技能开发指南]]、[[entities/agent-skill-writing-advanced|技能写作进阶]] 一致；而 [[entities/agent-skills-no-dependency-design-philosophy|零依赖技能设计哲学]] 主张技能零依赖，与本文"38 技能合包 + 协调者路由 + 依赖声明"的路线构成可对照的张力。harness 维度上，收益幅度随 harness 变化这一发现应与 [[concepts/harness-engineering|Harness Engineering]] 和 [[concepts/evaluation-harness-design|评估 Harness 设计]] 一起读；[[entities/agent-harness-skill-system-practical-guide|Harness 技能系统实践指南]] 与 [[entities/agent-skills-comprehensive-survey|技能综述]] 提供更宽的坐标系。^[raw/articles/hcls-agent-skills-domain-methodology-eval-2026.md]

→ [[raw/articles/hcls-agent-skills-domain-methodology-eval-2026|原文存档]]
