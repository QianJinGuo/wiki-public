---
title: "Ant SingGuard-NSFA: 蚂蚁开源AI Agent安全双模推理护栏框架"
created: 2026-07-12
updated: 2026-09-14
type: entity
tags: [agent, security, safety, ant, singguard, nsfa, open-source, claude-code, openclaw]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/claude-code-security-ant-singguard-nsfa-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Ant SingGuard-NSFA: 蚂蚁开源AI Agent安全双模推理护栏框架

> **Background**: 本文基于量子位对蚂蚁开源 AI 安全框架的报道建立。该框架包括面向智能体行为的 SingGuard-NSFA 和面向多模态大模型安全的 SingGuard 两个组件。

## 概述

2026年7月，蚂蚁集团开源了两套AI安全框架：**SingGuard-NSFA**（智能体安全）和 **SingGuard**（多模态安全），旨在将AI安全的焦点从传统的"内容审核"转向"行为安全"。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

这一开源背景是Claude Code和OpenClaw等Agent产品屡屡曝出安全漏洞（后门隐患、高危险漏洞），行业意识到单靠打补丁无法应对持续演变的风险。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

## SingGuard-NSFA: 智能体行为安全护栏

SingGuard-NSFA提供0.8B、2B、4B、9B四个尺寸，核心设计理念是将安全检查**前置到智能体执行之前**，在请求拦截和响应兜底两端同时设卡。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

### 风险分类体系

以经典的CIA三元组（机密性、完整性、可用性）为理论底座，结合三份OWASP大模型与智能体安全指南的实践经验，拆解智能体可能出现的风险。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

### 双模推理机制

框架采用两种模式同步进行风险拦截：^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

- **生成式模式**：逐条输出基于NSFA定义的链式推理分析，使每一步判断都有据可查，适用于离线合规审计
- **判别式模式**：每次前向传播直接给出各风险域的置信度，延迟可压到45-57ms，适用于高吞吐实时在线拦截

### 原生可扩展设计

骨干网络冻结，真正下判断的是外挂在上面的轻量分类头。出现新风险时只需补训一个小头，实现原生可扩展。可作为插件使用——为Llama Guard 3额外增加一个分类头后，用户请求安全基准的F1值直接提升17.6个百分点。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

### 评测表现

在3大评测基准（用户请求安全、模型响应安全、跨数据集泛化）上均取得SOTA。最小的0.8B模型就能比肩8B竞品，9B尺寸在泛化上达到91.29% F1，精度与召回更加均衡。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

## SingGuard: 多模态安全框架

面向多模态大模型，同样包括0.8B、2B、4B、8B四个尺寸。最大特点是把安全规则做成**运行时输入**——不同业务域可以现场下发各自的红线，模型据此逐条判定。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

推理侧采用**快慢分工**：快思考负责低延迟秒判，慢思考负责逐规则深度推理，两者之间通过early exit自动切换。针对线上多条规则并行审核的效率瓶颈，提出RI-Mask让共享的图文上下文只编码一次，多条规则并行判断，多模态推理最高可提速5倍以上。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

## 蚂蚁AI安全体系脉络

蚂蚁在AI安全上的布局逐步形成体系：从漏洞挖掘（发现多个OpenClaw高危漏洞），到场景化解法（与清华联合开源ClawAegis），再到可复用的底层框架。其智能体安全产品已通过信通院泰尔实验室最高等级评级。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

## 深度分析

### 双模推理：把"判定"与"解释"拆成两条独立通路

生成式模式用 SFT 习得链式推理，逐条对齐风险域，产出可审计的依据；判别式模式则让同一次前向传播直接映射为各风险域置信度，省去解码，延迟压到 45～57ms。离线审计要可追溯的理由，在线拦截要预算内的确定延迟，两条需求塞进同一条链路只会互相拖累。两条通路共享冻结骨干与同一套风险定义，因此"解释"不会漂移成事后编造的故事。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

### 冻结骨干 + 轻量分类头：把新增风险变成增量训练问题

风险分布持续漂移，"昨天合规、今天换个场景就踩线"是常态。骨干冻结、判定逻辑外挂在轻量分类头上，把新风险从"重训模型"降级为"补训小头"，不动既有检测能力，也省下全量重训的算力与能力回退风险。给 Llama Guard 3 加挂一个分类头，用户请求安全 F1 即提升 17.6 个百分点，说明护栏的瓶颈往往不在基座能力，而在风险空间没有被显式建模。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

### 单层提示词护栏为何失效，以及行为级拦截的差异

输入侧过滤只能看到"用户说了什么"，看不到工具调用的参数、文件读写路径与网络请求目标；而 Claude Code 的后门隐患与 OpenClaw 的高危漏洞，爆点都在**执行环节**而非内容环节。单层护栏把安全押在模型自身对齐上：上下文一旦被污染、工具返回值一旦被投毒，模型会在看似"合法"的推理链上做出越权动作。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

所以行为级拦截与输入过滤的差别不是强度而是位置：输入过滤是一道门，行为拦截是始终在线的旁路。前置拦截要在动作发生前给出判定，就必须与 Agent 执行循环耦合——请求侧拦下越界意图，响应侧对已生成的工具调用与执行结果兜底，单侧漏判时另一侧仍有收口机会，安全模型由此从"准入检查"转向"过程可控"。同类思路可参见 [[entities/agent-prompt-injection-defense-volcano-engine-2026|提示注入防御实践]]。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

### 多模态威胁模型与残留风险

SingGuard 把规则做成运行时输入，等于承认红线是业务变量：各业务域现场下发规则、模型逐条判定，回答的不只是"有没有风险"，还包括"是否违反当前防控规则"。推理侧快慢分工靠 early exit 自动切换，RI-Mask 让共享图文上下文只编码一次以支撑多规则并行，多模态推理最高提速 5 倍以上；威胁模型覆盖图像细节、图文组合与模型自身响应。残留风险在于：规则没写到的地方就是盲区，快思考的秒判在对抗性构造下可能被绕过，判别式只给置信度不给理由，误报排查成本被转嫁给运维。^[raw/articles/claude-code-security-ant-singguard-nsfa-2026.md]

## 实践启示

1. **闸门前置到工具调用之前**，在请求拦截与响应兜底两端设卡，而不是只审模型输出的文字。
2. **用"可解释 + 可增量扩展"两条标准挑护栏**：判定要能落回具体规则条目，新风险要能用补训轻量头吸收。
3. **优先给现有护栏加挂风险域分类头**——给 Llama Guard 3 加一个头即换来 F1 +17.6，性价比远高于换基座。
4. **把业务红线做成运行时配置**而非硬编码进权重，红线变化时现场下发比重训响应快得多。
5. **按延迟预算分流模式**：离线审计走生成式链式推理，在线高吞吐走判别式置信度（45～57ms 量级），共享同一套风险定义以免标准漂移。
6. **别把单层提示词护栏当终点**：提示注入、工具返回值投毒与越权执行分属不同层次，需要 [[entities/ai-agents-security-survey-attack-defense|系统性攻防视角]] 划边界，并参考 [[entities/amazon-bedrock-guardrails-code-generation-six-patterns|代码生成场景护栏模式]] 做分层设计。

## 相关实体

- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai|Agent安全三步序列]]
- [[entities/ai-agents-security-survey-attack-defense|AI Agent安全综述]]
- [[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2|AI工具投毒漏洞]]
- [[entities/anthropic-claude-code-trojan-telemetry-security-2026|Claude Code Trojan]]
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw架构]]

→ [[raw/articles/claude-code-security-ant-singguard-nsfa-2026|原文存档]]
