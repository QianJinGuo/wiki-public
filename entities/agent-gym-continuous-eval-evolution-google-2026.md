---
title: "Agent Gym：人机协同的 LLM Agent 持续评估与演化框架（Google Cloud）"
created: 2026-08-25
updated: 2026-10-01
type: entity
tags: [agent-gym, llm-agent, continuous-evaluation, agent-evolution, human-in-the-loop, rule-engine, constitution, external-correction, domain-agnostic, google-cloud, adk, spec-to-note, first-party]
rating: v7c8
sources: [raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026]
confidence: 0.85
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Agent Gym：人机协同的 LLM Agent 持续评估与演化框架（Google Cloud）

> Google Cloud 团队（Pouya Ghiasnezhad Omran 等）第一方论文（arXiv:2608.15591）。解决「静态智能体困境」：agent 行为在部署时冻结，而业务规则和边缘案例持续演化。Agent Gym 是模块化、领域无关框架，把任何现有 LLM agent 包进持续评估-演化循环，**在不修改 agent 源代码**的前提下实现持续评估、行为修正与演化。开源参考实现（发票处理，ADK + Gemini）。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md]

## 六个能力 × 三条架构区

六个可组合能力——**Act、Evaluate、Investigate、Correct、Learn、Observe**——组织在三个架构区：
- **Zone 1 宪法架构**：master data 规范（YAML）+ 重建规则书（Markdown）作为系统宪法，由领域专家手动编写或 bootstrap agent 生成
- **Zone 2 运行时推理管道**：acting agent 产出初始输出 → investigation agent 对照宪法校验 → ALF 引擎应用针对性修正
- **Zone 3 学习演化循环**：会话式学习 agent 让 SME 审阅、识别错误模式、通过程序化安全循环发现新修正规则

五个设计原则：Agent 是黑盒（零内部架构假设）、修正是分层而非侵入（独立下游层，可审计可回滚）、领域知识在配置而非代码、人类治理循环、治理必须分级（案例级 ALF 规则 vs 核心宪法修改分权）。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md]

## 核心机制：三层调查 + ALF 修正引擎 + 程序化安全循环

**Investigation Agent 三层调查架构**（无 ground truth 校验）：Layer 1 确定性检查（数据源校验+绕过检测，无 LLM）、Layer 2 LLM 规则发现（SHA-256 哈希缓存，后续运行摊销到零）、Layer 3 保守交叉验证（三重检查只报三次都确认的违规，降假阳性）。成本控制：内容哈希缓存/节过滤上下文/批量分组/三重检查早期退出，稳态调查成本由确定性检查主导。产出合规分 0-100%（≥80% 全合规 / 60-80% 部分违规 / <60% 重大违规暂停管道）。

**ALF 修正引擎**（检测与修正分离）：检测完全确定性——21 种条件操作符（相等/包含/正则/数值比较/列表成员/null/前缀/动态字段引用）AND 连接、完全可复现；修正三级动作——Tier 1 确定性字段编辑 / Tier 2 手术式修补（LLM）/ Tier 3 管道继续（LLM）。多规则时 Collect-Plan-Execute（Tier3→Tier2→Tier1 固定顺序，作用域互斥保证每 case 至多两次 LLM 调用）。每次修正完整审计。

**程序化安全循环**（关键创新）：在代码中强制而非 prompt，LLM 无法绕过。每条候选规则迭代校验：Schema 校验 → 目标匹配验证（失败则 LLM 保守拓宽）→ 附带影响评估（非预期匹配则自动加窄条件），循环最多三次，任何时刻不经过人工批准不应用。规则生命周期：enabled 标志/时间戳备份回滚/冲突检测/结构化元数据。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md]

## 宪法驱动的 agent 创建与 Spec-to-Note Gap

宪法 artifacts 同时是治理工具和开发规范（双向关系）：宪法即规范（LLM 可用作代码生成 grounding）、bootstrap 与精炼循环（constitution→code→refined constitution 倒转常规流程）、共同演化（修正规则揭示 acting agent 应更新处，宪法进化成活文档）、新 agent 上船（只需 agent + 宪法，计算组件领域无关）。

**Spec-to-Note Gap**：自编码器视角的 agentic 系统透明度——规范(自然语言)→实现(代码/Prompt/工具)→LLM 审计者→透明度 note(自然语言)，形成自然语言上的自编码器；对照 spec 与 note 作为重构损失，暴露缺失能力、静默作用域蔓延、评测套件从未测量的行为。SME 接口：SME 读不懂 harness 但能读结构化 note，评论成为系统 tickets。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md]

## 参考实现与局限

参考实现（发票处理，ADK+Gemini）：统一双模式 LlmAgent（18 个函数工具，推理+学习两模式），自包含 Python 模块。Acting pipeline 九阶段（Classifier→Extractor→Phase1-4 Validators→Transformer→Output Generator→Audit Logger），每阶段编号 JSON artifact 全可追踪。验证属性：领域适应性（换 master data YAML 即适配新文档）、运营就绪、自包含。

**局限**：bootstrap 目前手动；调查 agent LLM 成本随 case/规则组扩展（机制已大幅降低但量化是未来工作）；只在单一领域（发票）验证，多领域验证需确立领域无关性。**未来方向**：自动化规则建议、规则生命周期管理（晋升进 acting agent）、跨领域迁移、自动化 bootstrap、Spec-to-Note 作 release gate。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md]

## 深度分析

1. **修正与实现的解耦是这篇论文最核心的架构赌注**。Agent Gym 把「agent 行为出错」从代码问题重新定义为数据问题：错误模式沉淀为 ALF 规则（条件+动作的声明式记录），而不是直接改 Prompt 或代码。这带来一个可审计、可回滚、可批量分析的行为版本层——规则库本身就是 agent 的「补丁历史」。代价是双系统复杂度：21 种条件操作符的确定性引擎加上 LLM 修正路径，团队必须同时维护两套语义。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:60-70]
2. **成本工程是框架能否落地的隐形胜负手**。三层调查架构的每一层都内嵌成本对策：Layer 2 的 SHA-256 规则缓存把 LLM 规则发现摊销到零，节过滤让 LLM 只读规则书相关章节，三重检查对合规 case 单次调用即早期退出。这意味着稳态下系统的边际调查成本几乎与规则数量脱钩——这是「持续评估」从演示走向生产的关键工程决策，多数同类论文对此避而不谈。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:52-58]
3. **治理分级回应了 agent 运维中最容易被忽视的权限问题**。框架明确区分三类变更主体：案例级 ALF 规则由领域 SME 发现和批准，核心宪法修改需多利益相关者审批，acting agent 逻辑的永久变更走「规则晋升」路径。程序化安全循环（在代码而非 Prompt 中强制）保证 LLM 无法自我授权——这与「LLM 自我改进」路线形成对照：这里演化的每一次推进都有人类签名，LLM 只负责提议和起草。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:42-43,70]
4. **Spec-to-Note Gap 把透明度问题转化为一阶信号处理问题**。将 spec→实现→note 视为自然语言自编码器，重构损失（spec 与 note 的偏差）直接指向未测量行为、静默作用域蔓延和缺失能力。其深层洞察是：审计输出必须与 SME 的认知接口对齐（可读的自然语言 note），否则评测数据再多也无法进入治理循环——SME 的注意力才是整个系统的稀缺资源。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:78-79]
5. **宪法双向流（constitution→code 与 code→constitution）让框架超越了评估工具**。正向流用宪法作为代码生成的 grounding（bootstrap agent 四阶段流水线），反向流让运行时积累的修正规则反哺宪法演化为活文档。这与「规格文档写完即过时」的常见宿命相反：宪法在这里有持续的经济激励保持更新，因为它是修正规则的挂靠点和规则晋升的目标态。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:33,75-76]

## 实践启示

1. **给生产 agent 加一层外部修正层，而不是改 Prompt**。当 agent 出现系统性误用时，先用「条件→动作」规则记录错误模式并在下游修正（保留原始输出可审计），累积验证后再晋升进 agent 永久逻辑——避免每次纠错都重部署、丢失可回滚性。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:40-43,89]
2. **把领域知识写成机器可解析的宪法（YAML master data + Markdown 规则书），使其可被校验和缓存**。这样新领域只需换配置，校验规则可哈希缓存摊销成本，且同一份宪法可直接用作新 agent 代码生成的 grounding——一份资产三处复用。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:33,41-42,75-76]
3. **无 ground truth 的校验用「确定性优先 + LLM 保守兜底」的分层设计**：纯代码检查（数据源/绕过检测）打底，LLM 规则检查结果做三重交叉验证且偏保守（模糊判合规），假阳性会被 SME 惩戒机制放大，宁漏勿错。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:52-58]
4. **任何 LLM 产出的规则必须过程序化安全门**：schema 校验→目标匹配→附带影响评估，对历史 case 确定性求值，意外匹配自动加窄（vendor/金额范围/类别等条件），循环封顶三次、超限带警告交人工。这是防止「一条宽泛规则静默污染大量正常 case」的最小可行护栏。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:70]
5. **为 SME 设计与 harness 解耦的审阅接口**：把 agent 行为编译成结构化自然语言 note（Spec-to-Note 模式），让领域专家标记错误假设和监管边缘案例，其评论直接转为系统 tickets——评测闭环的瓶颈在人而不在指标。 ^[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026.md:78-79]

## 关联实体

- [[entities/self-evolving-agents-survey|自演化 Agent 综述]] — agent 演化的系统综述
- [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六机制]] — agent 自我改进机制维度
- [[entities/hermes-agent-self-evolution-tengxun|Hermes 自演化（腾讯）]] — 自演化 harness 实践
- [[entities/langsmith-engine-self-improving-agent-trace-based|LangSmith Trace-Based 自改进]] — 基于 trace 的 agent 自改进
- [[raw/articles/agent-gym-continuous-eval-evolution-google-paper-2026|原文存档（论文）]]
- [[raw/articles/agent-gym-continuous-eval-evolution-xhs-2026|原文存档（解读号）]]
