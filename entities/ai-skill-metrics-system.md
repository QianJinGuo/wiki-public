---
title: "AI Skill 测评指标体系"
created: 2026-05-14
updated: 2026-11-20
type: entity
tags: [ai-skill, evaluation, metrics, testing, agent, llm, quality-assurance]
sources: [raw/articles/ai-skill-测评指标体系]
review_value: 8
review_confidence: 8
review_recommendation: strong
review_stars: 4
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 指标体系与02章重复; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---
# AI Skill 测评指标体系

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/ai-skill-metrics-system.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/regression-tax-skills-hurt-llm-agents|Regression Tax: 技能包导致 Agent 性能退化的系统性分析]] — Regression Tax：5832实验，回归抵消59%毛增益
- [[entities/autoresearch-taxonomy-chengzihong-chengzihong|AutoResearch 分类法：四种 Agent 循环设计与四维分析框架]] — 四循环四维分类法
- [[entities/what-comes-next-with-open-models|开源模型的下一阶段：三类模型分类与生态化路径]] — 三类模型分布智能论
- [[entities/gpt-54-is-a-big-step-for-codex|GPT 5.4 是 Codex 的一次大跨越：四维评估视角与 Agent 战争回归]] — 四维评估+Claude/GPT哲学分歧5698字全版
- [[entities/taobao-smart-shopping-guide-agent-evaluation-pzmx|多轮 Agent 场景下，滴滴的 EAGLE-3 训推加速实践]] — EAGLE-3训推加速实践
- [[entities/ai-cambrian-google-agentic-rag-sufficient-context-cross-corpus-20260606|Google Agentic RAG 跨语料库框架：充分上下文智能体 + 5 阶段管线]] — 充分上下文RAG框架

## 工程实践
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/ai-skill-测评指标体系|ai-skill-测评指标体系]] — 九层指标全版
- [[entities/ai-skill-skill-creator-源码拆解|05—skill-creator 源码深度拆解：LLM Skill 触发率、防过拟合与三 Agent 评审完整指南]] — 源码级测评实现
- [[entities/anthropic-95pct-data-analysis-jiagoux-data-level-harness-20260606|数据级 Harness：架构师 JiaGouX 解读 Anthropic 95% 数据分析与 5 个反直觉边界]] — 数据级harness解读
- [[entities/autoresearch-marketing-growth-amap-ai-native|高德 Marketing AutoResearch：AI Native 营销增长经营托管框架]] — 营销经营托管
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation-framework|AgentEval：YAML驱动的Agent评测框架]] — 评测框架全版
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进|存之有序，治之有矩——Agent 记忆系统的工程实践与演进]] — 写入纪律prompt cache冲突
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps: Operationalize agentic AI at scale with Amazon Bedrock AgentCore]] — 四支柱解析版
- [[entities/你不知道的-agent原理架构与工程实践-v2|你不知道的 Agent：原理、架构与工程实践]] — 真实内容harness影响实证
- [[entities/anthropic-institute-when-ai-builds-itself-jiagoux-interpretation|Anthropic Institute《When AI builds itself》深度解读：AI 进入 AI 研发执行层、瓶颈迁移与研发级 Harness（架构师 JiaGouX）]] — 解读短条borderline
- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|Agent-EvalKit：AWS 开源 CLI Agent 评测工具包]] — 评测工具包设计
- [[entities/management-as-ai-superpower|Management as AI superpower]] — delegation三变量公式+GDPval 72%实证
- [[entities/hermes-agent-long-running-governance-five-cards-ruofei|长期运行的 Agent 怎么管：Hermes 治理分层与 5 张卡]] — don't automate slop+5张卡治理
- [[entities/tencentdb-agent-memory-context-offloading|腾讯云Agent Memory：Mermaid无限画布×上下文卸载]] — Mermaid画布上下文卸载
- [[entities/langsmith-trajectory-evals|Langsmith Trajectory Evals]] — 轨迹评估四匹配模式
- [[entities/secure-ai-agents-with-policy-and-lambda-interceptors-in-amaz|Secure AI agents with Policy and Lambda interceptors in Amazon Bedrock AgentCore gateway]] — Cedar策略+Lambda拦截器双模式
- [[entities/anthropic-long-running-agent-architecture-6h-retroforge|Anthropic 实战分享：如何让 AI Agent 持续工作几天？]] — 长时运行对抗架构
- [[entities/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606|小刘商业 Agent 增强层通用基座]] — 基座+增强层论点短条
## 深度分析

### 增益 Δ 是整个体系的「第一否决权」
原文最反直觉的一点：通过率再高也可能掩盖 Skill 无用甚至有害的事实。Δ 定义为 with_skill 与 without_skill 两条基线通过率之差，只有 Δ < 0 才是硬红线，但 Δ ≈ 0 同样危险——SkillsBench 的 84 个有效任务中约 19% 出现负向增益，说明「加了 Skill 就变好」在真实任务分布上并非默认成立。三条件对比范式（纯模型基线 / 人工 Skill / 自生成 Skill）把判断进一步变成可操作的：人工版跑不过自生成版，往往意味着人工 Skill 过度约束了模型。这与 [[entities/regression-tax-skills-hurt-llm-agents|Regression Tax]] 的结论形成呼应——Skill 的隐性成本不只是幻觉与延迟，还有对既有能力路径的覆盖。^[raw/articles/ai-skill-测评指标体系.md:60-72]

### 通过率 × Δ 矩阵与准入表的组合判断
原文用「通过率 × Δ」二维矩阵给出四种处置：高/高正常发布，高/≈0 评估 Skill 是否有存在必要，低/>0 方向对但执行有问题、继续优化，低/<0 停止发布；再补一条「高通过率但覆盖率 < 50% → 补充用例」。这个矩阵的价值在于把发布决策从单指标判断升级为组合判断——单一高通过率在低覆盖下毫无说服力。而完整准入表把九层维度压成一张可执行清单：S 级要求通过率 ≥95%、IFR = 100%、幻觉 0 次、灾难场景全过、P95 < 15s。工程上可参照 [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|Agent-EvalKit]] 的做法把这张表直接编译进 CI 门槛。^[raw/articles/ai-skill-测评指标体系.md:99-111]

九层维度的记忆口诀「触发→输出→规则→对话→容错→效率→设计→覆盖→维护」也不只是记忆术，它同时编码了测试顺序与成本曲线：三层内圈（触发、输出、业务）任何模式都必测，中层（交互、健壮、工程）在 standard 起进入，外圈效率与设计只有 full 模式必测，组织层仅是建议。资源紧张时砍掉外围层是体系允许的降级路径，但内圈层是不可让步的底线——模式选择（quick/standard/full）本质是按 Skill 风险等级购买证据深度的决策。

### 指标分层背后是「证据等级」的自觉
原文刻意区分了两类指标来源：通过率、Δ、触发率、幻觉检测、稳定性分别能追溯到 OpenAI Evals/HELM、SkillsBench 论文、信息检索的 Recall/Precision、TruthfulQA/FactScore 与统计学标准差；而 S 级通过率 ≥95%、IFR = 100%、Stddev < 0.05 这些具体阈值没有学术背书，是按业务容错度、用户预期、历史数据三因素拟定的经验值。这个区分让体系可以被「批判性使用」：结论性指标可信，阈值可按业务调整。照搬阈值而不审视其来源，是团队落地这套体系时最常见的误用。^[raw/articles/ai-skill-测评指标体系.md:126-137]

### INCONCLUSIVE 与触发率：两个最易误读的数字
INCONCLUSIVE 不是失败，而是测试资产未就绪（文件类型不符、账号权限不足、测试方法绕过 Skill 层、数据过期），不计入通过率但必须在报告里给出补充计划——把它当 FAIL 会让通过率虚低，当 PASS 又会掩盖资产缺口。触发率则天生带统计近似：AI 模拟方案只用 10 条 prompt（5 应触发 + 3 不应触发 + 2 边界）估算，TP ≥ 80% 只是参考值而非硬性 FAIL 条件；同时 Claude 有 undertrigger 倾向，Description 需要写得「pushy」一些，明确写出未明说场景也应触发。这两个指标共同提醒：体系里的每个数字都有适用边界，脱离生成方式谈数值没有意义。^[raw/articles/ai-skill-测评指标体系.md:23-30]

## 实践启示

1. 发布前先算 Δ 再看通过率：任何 Skill 上线前跑一次 without_skill 对照基线，Δ < 0 直接否决，Δ ≈ 0 重新评估存在必要性，避免为无增量的 Skill 付出长期维护成本。
2. 把阈值当可调参数而非铁律：S 级 ≥95%、Stddev < 0.05 属于经验值，可按业务容错度调整，但要在团队内显式记录调整依据；学术来源指标（通过率、Δ、触发率）保持原定义以保证跨团队可比性。
3. 为 INCONCLUSIVE 建立补充资产清单：报告单独列出未决用例及其缺口（文件类型 / 权限 / 数据时效），下一轮测试前补齐重跑，而不是让它静默消失在通过率里。
4. 对 Claude 类模型把 Description 写得「推一把」：显式加入「即使用户没有明确说，遇到 X 情况也要使用」来对抗 undertrigger；用 10 条 AI 模拟 prompt 快速估算触发率，低于 80% 先改描述再重测。
5. 用 IFR 补通过率的盲区：通过率 92% 可能藏着 20% 的硬性规则违反；涉及资金、审批的 S 级流程要求 IFR = 100%，把「必须 / 禁止 / 固定为」类规则单独抽出统计。
6. 落地时优先复用现成评测基建：[[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation-framework|AgentEval]] 与 [[entities/langsmith-trajectory-evals|Langsmith Trajectory Evals]] 已覆盖 YAML 用例驱动与轨迹级评估，可将本文九层指标接入既有评测流水线而非自研采集。

→ [[raw/articles/ai-skill-测评指标体系|原文存档]]
