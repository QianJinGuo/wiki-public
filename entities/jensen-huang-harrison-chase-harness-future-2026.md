---
title: 黄仁勋 × Harrison Chase 对话：未来公司将建立在 Harness 之上
created: 2026-07-16
updated: 2026-09-18
type: entity
tags: [harness, agent, nvidia, langchain, jensen-huang, enterprise-ai, agentic-system, open-source]
status: verified
confidence: 0.9
provenance_state: extracted
sources: [raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> NVIDIA CEO 黄仁勋与 LangChain CEO Harrison Chase 的 26 分钟深度对话（2026-07），阐述了未来企业将由 Harness 系统而非传统业务流程定义的愿景。对话涵盖开源 vs 前沿模型策略、企业智能内建、Agent 安全治理、以及从编程到构建 Agent 的范式转变。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

## Harness 作为企业新基石

黄仁勋的核心论断："今天的多数公司建立在'业务流程'之上，未来的公司将建立在 **Harness** 之上。" LangChain 将成为创建公司"操作系统"的工具，将传统工作流转变为自主、智能、高效的 Agent 系统。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

## 开源权重模型 vs 前沿模型策略

Harrison Chase 披露 Nemotron-3-Ultra 在 DeepAgents 中加入 LangChain 优化后，内部基准达 **86% 准确率**（Claude Opus 87%），成本仅为 Opus 的十分之一。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

黄仁勋的策略：
1. **先从前沿模型开始**——了解能力天花板
2. 再打造专业化超级智能体（Super Agents）——连接到专门工具，专精于单项任务
3. 很多东西可能永远不需要替换，因为前沿模型自身不断进步

## 企业智能必须内建

"当企业需要增强自己的智能时，不能指望给第三方打个电话解决，你需要就在公司内部完成。" 每家公司都建立在某种专业化的智能（知识产权）之上。通用技能可用通用模型，但**核心智能不能外包**。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

## 从编程到构建 Agent 的范式转变

黄仁勋：NVIDIA 软件工程师**更愿意构建智能体而不是写 Python 代码**。写代码就像打字，他们现在成为系统工程师，创建评估系统、基准测试和护栏系统。将 AI 引入现实世界的工作量巨大，创造了大量新岗位。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

## Agent 安全与治理

部署 Agent 需要配套的安全机制和访问控制。AI 也需要一套类似于**HR 的系统**：每个 Agent 有使命文档、工具访问权、信息访问权限，IT 组织能在公司内部构建、改进和部署这些智能体。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

## 深度分析

### Harness 而非模型，正在成为企业的真正护城河

黄仁勋的判断把"模型能力"从竞争焦点上挪开：基础智能正在变成可采购的商品，难以复制的部分是包裹在模型外面那一层——工具、记忆与检索、评估、编排、权限。同一块权重放进不同的 Harness，表现差异可以覆盖乃至超过模型代际差：Harrison Chase 披露在 DeepAgents 中优化提示词与工具调用方式后，Nemotron-3-Ultra 的内部基准达到 86%，逼近 Claude Opus 的 87%，成本却只有后者的十分之一 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:40]。这说明"换模型"的边际收益正在被"改 Harness"追上甚至反超，护城河从权重转移到其外围系统。参见 [[concepts/harness-as-product-surface|Harness 作为产品面]]。

### 开源权重与前沿模型的经济学：成本、延迟、主权三角

对话里最值得拆解的不是"谁更准"，而是那组数字的搭配：1 个百分点的准确率差，换来十分之一的价格 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:40]。黄仁勋进一步把低成本的战略意义说清楚——智能变便宜之后，使用频率会大幅上升，团队可以在更大的搜索空间里迭代，而极快的推理速度让模型能反复试错、逼近更优解 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:42]。所以企业选型不是"选更强的模型"，而是"选单位预算下能跑多少轮迭代的模型"：一次调用便宜十倍，如果只调用一次，价值等于零。真正的约束是三者同时权衡——采购与推理成本、交互延迟、数据与合规主权（能否本地部署、专有知识能否不外流）^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:74]。参见 [[concepts/ai-cost-optimization-framework|AI 成本优化框架]]、[[concepts/local-vs-cloud-agent-deployment-strategy|本地 vs 云端部署策略]]、[[entities/25-the-unbearable-cheapness-of-open-weight-models|开放权重模型的廉价化]]。

### "内建智能"的逻辑：为什么核心智能不能外包

"当企业需要增强自己的智能时，不能指望给第三方打个电话解决" ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:58]。这条论断的推论是分层采购：编程、写作这类通用技能交给通用模型即可；而构成企业 IP 的专业判断必须长在内部。黄仁勋同时给出一条可操作的路径——先从能力最强的前沿模型起步，摸清天花板；再把它拆成连接专用工具的超级智能体（Super Agents），每个只专精一件事；最后才判断哪些环节值得换成自有或开放权重 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:46]。关键在于"替换"不是默认动作：前沿模型自身仍在进步，很多环节可能永远不需要自建 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:52]。NVIDIA 内部正是用 DeepAgents 加 Nemotron-3 处理供应链优化、芯片设计优化这类高复杂优化问题 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:48]。参见 [[concepts/enterprise-ai-adoption|企业 AI 采纳]]。

### 从写代码到监督智能体：一次劳动模型的变化

"写代码就像打字" ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:64]。这句话描述的不是失业，而是岗位内容的迁移：NVIDIA 的软件工程师更愿意构建智能体而不是写 Python，他们新的产出物是评估系统（Evals）、基准测试与护栏系统 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:66]。工程师的角色从"实现逻辑的人"变成"定义成功标准并守在循环外面的人"——把 AI 带进真实世界的工作量极大，因此这里产生的是新岗位而非净减员。对个人的含义是能力重心转移：会写代码不再是稀缺技能，能把模糊业务目标翻译成可评测的成功信号才是。参见 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测与基准框架]]、[[concepts/agent-orchestration-patterns|Agent 编排模式]]。

### 治理属于 Harness 层，而不是模型层

"AI 也需要一套 HR 系统"这个类比的落点很具体：每个 Agent 要有使命文档、工具访问权、网络与信息访问权限，IT 组织要能在公司内部完成这些智能体的构建、改进与部署 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:70]。权限、审计、评测这些机制天然长在模型之外：模型只负责在给定边界内推理，谁可以调用什么工具、可以读哪些数据、什么行为算越界，全部由 Harness 裁定。这带来一个直接的工程后果——安全投入应当优先落在外围系统的可观测性与最小权限设计上，而不是期待模型自身"更听话"。参见 [[concepts/agent-security-architecture|Agent 安全架构]]。

## 实践启示

1. **先把 Harness 立起来，再谈换模型。** 在评估任何模型替换之前，先固化工具接口、记忆与检索、评估集、权限边界——同一模型在不同 Harness 下的差距，可能大于模型之间的差距 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:32]。
2. **用"单位成本能跑多少轮迭代"做选型，而不是"单次调用质量"。** 开放权重模型的十倍成本优势，只有被 Harness 放大成搜索空间与迭代次数时才真正兑现 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:42]。
3. **先从最强前沿模型起步，再逐步内化。** 借顶级模型探清能力天花板，再识别哪些环节值得换成自有或开放权重；同时接受"很多环节永远不必替换"这一前提 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:44]。
4. **把核心智能留在公司内部。** 通用技能可以外购，构成知识产权的专业判断必须自建，至少保证对权重或 Harness 的控制权在自己手里 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:58]。
5. **给每个 Agent 发一份"入职资料"。** 使命文档、工具清单、数据与网络访问范围、可回滚的部署路径——缺一则无法真正投产 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:70]。
6. **把工程重心从写代码转向写评估与护栏。** 让工程师的产出物变成 Evals、基准与安全边界，这是把 AI 引入真实世界时真正稀缺的工作 ^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md:66]。

## 关联条目

- [[entities/nvidia-nemotron-3-ultra-sagemaker-jumpstart-moe-agentic|NVIDIA Nemotron-3-Ultra]] — 对话中提到的模型，开源权重达到前沿性能
- [[entities/nvidia-secure-local-agent-nemoclaw-openclaw|NVIDIA 安全本地 Agent：Nemoclaw/OpenClaw]] — NVIDIA 的本地 Agent 运行环境（NIM）
- [[entities/nvidia-agentic-systems-extreme-co-design|NVIDIA Agentic Systems Extreme Co-Design]] — NVIDIA Agent 系统的另一次深度技术阐述
- [[entities/deep-agents-bedrock-agentcore-subagent-orchestration-aws|DeepAgents — AWS Bedrock AgentCore 子智能体编排]] — 对话中提及的 DeepAgents 框架的实践

## 退出

→ [[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16|原文存档]]
