---
title: "SkillOS: Learning Skill Curation for Self-Evolving Agents"
created: 2026-05-12
updated: 2026-11-17
type: entity
tags: [agent-tools, ai-agent, llm, newsletter, agent]
review_value: 7
sources: [raw/articles/skillos-learning-skill-curation-for-self-evolving-agents]
review_confidence: 8
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.78: newsletter抓取含模板噪音; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# SkillOS: Learning Skill Curation for Self-Evolving Agents

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.78**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/skillos-learning-skill-curation-for-self-evolving-agents.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/llm-agent脚手架如何具备自进化能力以hermes-agent为例|LLM agent脚手架如何具备自进化能力？——以hermes agent为例]] — Hermes自进化15402字最全版
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent|Hugging Face AI Agent 术语表：Model / Agent / Scaffolding / Harness / Context Engineering / Policy / Tool / Skill / Sub-agent 完整区分]] — HF术语表16399字：Scaffolding/Harness/Policy辨析
- [[entities/agent-skills-comprehensive-survey|Agent Skills 系统性综述：表示→获取→检索→进化]] — skill综述三元组
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/z-glm-5.2|GLM-5.2: Built for Long-Horizon Tasks]] — GLM-5.2长程任务发布
- [[entities/morphllm-codegen-inference-optimization|Optimizing Models to Be Fast at Codegen]] — codegen locality复用+Chinchilla在投机解码失效
- [[entities/foundation-capital-agent-era-six-insights|Foundation Capital Agent Era Six Insights]] — Agent时代六条投资洞察rv9框架
- [[entities/perplexity-search-as-code-generation|Rethinking Search as Code Generation]] — Search as Code：查询变可执行代码对象7660字rv9
- [[entities/deepseek-visual-primitives-thinking|DeepSeek Thinking with Visual Primitives 深度解读]] — 视觉原语10k主版
- [[entities/mira-mpa-deep-principle-ai4s-40-sota|MIRA + MPA：深度原理 AI Scientist 递归自训练打造材料基座模型，40 项实验全面 SOTA]] — AI Scientist递归自训练，35/40胜前SOTA
- [[entities/gpt-5-6-preview|GPT-5.6 Preview System Card — Community Detection & Benchmarks]] — GPT-5.6系统卡：三模型+安全评估rv9

## 工程实践
- [[entities/karpathy-vibe-coding-agentic-engineering-v4|Karpathy 最新访谈：从 Vibe Coding 到 Agentic Engineering]] — v4 8090字rv10：可验证性上限+MenuGen警示
- [[entities/harness-engineering|'Harness Engineering：AI 从]] — 六层架构+七大反模式+分级决策树19712字rv9
- [[entities/building-multi-tenant-agents-with-amazon-bedrock-agentcore|Bedrock AgentCore 多租户 Agent 构建实践]] — 十组件多租户
- [[entities/build-ai-agents-for-business-intelligence-with-amazon-bedrock-agentcore|Bedrock AgentCore 构建 BI 智能体]] — BI三agent案例
- [[entities/andrej-karpathy-claude-md-134k-stars-2026|最佳 Claude Code 配置：Andrej Karpathy 的 CLAUDE.md，134+k star了！]] — CLAUDE.md四规则解析
- [[entities/yidian-tianxia-context-engineering-agentic-ai-qcon|一点天下：Context Engineering 与 Agentic AI (QCon)]] — 7114字最全六层上下文版
- [[entities/integrating-aws-api-mcp-server-with-amazon-quick-suite-using-amazon-bedrock-agen|AWS API MCP Server + Quick Suite + Bedrock AgentCore 集成]] — NL→AWS API范式+AgentCore认证模型
- [[entities/break-the-context-window-barrier-with-amazon-bedrock-agentcore|Bedrock AgentCore RLM：突破上下文窗口限制]] — RLM程序化环境
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/ai-native-dan-shipper-every-layered-thinking-walkwalk|AI Native 公司 Dan Shipper 30 人做出 100 人产出：层次化思维 + SaaS 6 维能力框架]] — 30人100人产出
- [[entities/minimax-agent-team-mavis-owner-worker-verifier|一个 AI 还是不够的：MiniMax Agent Team（Mavis）]] — 四痛点+对抗式验证7527字全版
- [[entities/build-ai-powered-dashboard-automation-agents-with-nlp-on-amazon-bedrock-agentcor|Bedrock AgentCore NLP 仪表盘自动化 Agent]] — 仪表盘三代理
- [[entities/claude-code-source-leak-lifecycle-analysis|CLAUDE.md]] — 8步生命周期10k


## 深度分析

### 学习式技能策展：从"存什么"到"怎么管"
SkillOS 的核心立场是：自进化 agent 的瓶颈不在记忆是否存在，而在记忆如何被治理。它把技能库当作需要持续运营的资产，用独立于执行器的 Skill Curator 在每条轨迹结束后做三步——分析轨迹成败并归因、提取可复用知识（成功轨迹提炼技能，失败轨迹提炼避坑技能）、再对技能库执行插入 / 更新 / 删除三种工具调用。值得注意的是删除与插入同为一级操作，技能库因此不是只增不减的日志，而是被持续修剪的知识集合；同时 curator 被要求把轨迹压缩为可泛化的程序性知识（具体对象 ID 替换为变量），而非原样存档。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:13-46] 这种"轨迹→技能"的蒸馏思路与 [[entities/trace2skill-trajectory-distillation-agent-skills|Trace2Skill：轨迹蒸馏为 Agent 技能]] 一脉相承，但 SkillOS 把蒸馏本身做成了可训练的策略。

### 用 RL 训练"图书管理员"：奖励来自下游任务
最反直觉的设计是把 curator 当作可优化对象：用 GRPO 训练策展策略，奖励一部分来自外部 judge（Qwen3-32B）对技能质量的四维评估——抽象性、可复用性、可操作性、忠实性，另一部分关联任务本身的成败信号。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:72-90] 消融结果很有说服力：换用 Gemini-2.5-Pro 做免训练 curator 的 SkillOS-gemini，与同骨干但不做 RL 的 SkillOS-base，都输给了 RL 训练后的开源小模型（平均成功率 73.1%）——学习 curator 带来的增益大于换更强的 curator 骨干；手工启发式策展的 MemP 甚至低于无记忆基线，说明存取规则本身是易碎的组件。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:260-263,290-295] 与 [[entities/skillos|SkillOS 技能库体系]] 相关的工程结论是：治理逻辑应当从数据中学习，而不是拍脑袋写规则。

### 数据分组："相关但不冗余"的课程带宽
训练数据不是随机任务流，而是两阶段构造的任务组：先给每个任务标注五维短语属性（主题、所需技能、概念定理、启发式策略、常见陷阱），再用 soft-Jaccard 相似度加依赖门控挑选后继任务。门控的六个条件中最关键的几条：必须与源任务共享基础概念与技能（保证新技能有复用场景）、整体相似度必须落在"既非近重复又不太无关"的区间（保证有新东西可压缩）、难度必须前向递进（形成课程信号）。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:172-174] 这本质上是在为 curator 制造最优学习压力：任务太像则无事可学，太远则技能无法沉淀复用。

### 案例揭示的技能形态学与表达力边界
案例分析显示 curator 学会了按任务结构塑造技能形态：agentic 任务产出失败恢复的元策略（穷尽搜索→确认不可得→寻找替代品→继续执行），并显式引用已有技能形成组合；推理任务产出"分支式"技能，单条技能编码多条解题路径，各带公式与前置约束，而非逐字复制的轨迹。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:295-297,303-304] "看桌灯下的 CD"案例说明好技能的作用是收窄探索空间，而非背诵动作序列。局限同样清晰：技能被简化为单文件 Markdown（跟随 [[entities/anthropic-agent-skills-design-patterns-14|Anthropic 14 个 Agent Skills 设计模式]] 所代表的 SKILL.md 范式），丢失了可执行脚本与层级组合两种表达力；检索停留在 BM25 关键词匹配；执行器全程冻结，技能与执行器的错配成本全由 curator 一侧吸收。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:306-316] 论文自己也把 agentic 检索、层级化组合技能、多智能体共享记忆列为下一步方向。^[raw/articles/skillos-learning-skill-curation-for-self-evolving-agents.md:331-338]

## 实践启示
1. **给技能库配治理动词集**：insert / update / delete 三操作缺一不可，删除尤其不能省——过时与重复技能会污染检索，只增不减的技能库迟早退化为噪音堆。
2. **用四维清单审技能质量**：抽象性（去实例化）、可复用性（原子模块化）、可操作性（有工作流与不适用条件）、忠实性（不虚构轨迹外事实），可直接移植为人工或 LLM 评审的评分卡。这与 [[entities/agent-skills-no-dependency-design-philosophy|Agent Skills 无依赖设计哲学]] 中"原子、自包含"的原则互相印证。
3. **学习策展优于堆模型**：与其换更强的模型做技能维护，不如让策展逻辑接受任务成败反馈的持续校准；手工启发式规则在执行器变弱时最先崩溃。
4. **构造训练/评测任务组时用"相关但不冗余"门控**：共享基础、非近重复、难度递进三条件并行，比随机配对或纯主题聚类都更能逼出可复用技能。
5. **让技能形态跟随任务类型**：程序性任务存可组合的工作流元策略，推理任务存多路径加前置约束的分支式条目；单一模板通吃会损失针对性。
6. **承认单文件 Markdown 的表达力上限**：声明性知识用 SKILL.md 足够，可执行过程应外挂脚本、复杂能力应允许技能间组合引用，否则只能压扁成散文。
