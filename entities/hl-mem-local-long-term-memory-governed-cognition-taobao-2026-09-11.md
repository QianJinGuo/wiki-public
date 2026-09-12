---
title: "HL-Mem：本地长期记忆系统 — 事件溯源式记忆治理（大淘宝技术）"
created: 2026-09-11
updated: 2026-09-11
type: entity
tags: [agent-memory, long-term-memory, hl-mem, memory-governance, cognitive-architecture, event-sourcing, bitemporal, sqlite, local-first, chinese-nlp, taobao, alibaba, open-source]
sources:
  - raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11
confidence: 0.8
provenance_state: extracted
---

> → [[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md|原文存档]]

# HL-Mem：本地长期记忆系统 — 事件溯源式记忆治理

HL-Mem（github.com/lohr13/hl_mem，Apache-2.0，v1.1）是淘天集团-直播AIGC团队（作者子安）开源的本地化 Agent 长期记忆系统，核心命题：把记忆当作**受治理的认知系统**而非数据存储——记忆的本质是在检索（RAG）之上加治理层：去重、冲突消解、生命周期、可追溯。默认部署仅需一个 SQLite 文件（WAL + FTS5 + 向量 BLOB）加一个后台 Worker，`pip install hl-mem` 即装 ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

## 双通道认知数据模型

**事实通道**（维护认知）：`Event（不可变原始经历）→ Claim（原子事实：主体/谓词/值/时间/重要性）→ Observation（观察）→ Mental Model（心智模型）`。Evidence Link 把 Claim 指回原始 Event——模型生成的文字不能充当事实来源；底层事实失效时派生理解被标记为需重算 ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

**经验通道**（积累经验）：`Event → Episode（任务级经验）→ Trace/Reward（过程与反馈）→ Policy（选择策略）→ Procedure/Skill（可复用做法）`。两条通道共享 Event 起点、按不同规则演化：事实被新证据纠正，经验被反馈强化或淘汰。作者论点：把两者压成同一种"记忆文本"，系统既解释不了为什么相信一件事，也解释不了为什么选择一种做法 ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

## 五个关键设计取舍

**① 先存案发现场，再允许形成看法**：Event 不可修改，Claim 只是系统对 Event 的解读——任何记忆错误可诊断可修复。v1.1 加入 `origin_class`/`session_kind` 来源权威分级：用户直述、外部网页、cron、heartbeat、subagent 不再默认拥有同等事实权威，`provenance.mode=enforce` 在调用模型前拦截不适合自动成 Claim 的来源（把提示词注入挡在长期认知之外） ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

**② supersede 链 + 双时间模型**：纠正不是 UPDATE——新 Claim 经 supersede 链替代旧 Claim，旧值退出召回但留在历史/审计。同时记录 valid time（现实何时为真）与 recorded time（系统何时知道）：3 月说在杭州、8 月说 5 月已搬北京，系统既能答"4 月时以为你在哪"（杭州，当时认知）也能答"4 月实际在哪"（北京，事后修正） ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

**③ 去重是漏斗不是阈值**："部署 v1.2" vs "部署 v1.3" 余弦相似度极高但语义相反。漏斗：精确哈希 → 确定性安全门检查 protected atoms（主体/数字/版本/日期/路径/极性/引语/姓名/实体，须全部一致才可合并）→ 灰区才交给 LLM 审计。召回侧 `entity_constraint_mode=enforce` 隔离同名实体——冻结门禁中 15 个高置信实体案例全部进 Top-5、跨实体错误 Top-1 从 15 降到 0。倾向保守偏差：多留重复只浪费空间，错误合并会永久改写事实 ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

**④ 遗忘是新陈代谢不是 Delete API**：三层递进——自然衰减（TTL + importance 半衰期激活度，只影响"多容易被想起"不改写置信度）→ 归档（退出召回、清向量、事实留历史可查证）→ 显式遗忘（沿依赖关系物理删除闭环 + 派生理解失效 + tombstone 账本阻止备份恢复复活旧记忆）。作者定位：被遗忘权与本地隐私场景的硬需求，裸 RAG 难补齐 ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

**⑤ 让 LLM 少跑几趟**：同一 Session Event 形成有界窗口（默认最多 5 条、最长等 120 秒）一次性结构化抽取；去重/冲突/维护确定性规则优先。**关系发现默认关闭**——图谱不是免费索引：抽取/消歧/合并/重算成本组合膨胀，一条错误关系会沿多跳检索扩散成连锁污染；关系写入视为高风险变更，候选关系需显式开启+审核。`hl-mem ops report` 把调用量/Token/费用/延迟/失败率变成可观测预算约束（无法定价时 fail-closed） ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

## 五层架构与中文一等公民

接入层（CLI/REST/MCP/Hermes Provider）→ 写入层（可持久化 Job 事务提交，失败重试，不提前发布"已记住"）→ 认知层（双通道）→ 召回层（中文 FTS5 + Embedding + RRF 融合，实体约束隔离，新近度/重要性/反馈效用排序，Token 预算 Context Packet）→ 治理层（后台 Worker 扮演"睡眠"：去重收敛/冲突归并/TTL 清理/归档异步执行）。中文路径：Jieba 领域词典预分词全文索引（通用 FTS 对中文近乎失效），原文不翻译、语义中英双语对齐；模型层兼容百炼/DashScope、智谱、OpenAI-compatible ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

## 认知科学翻译表

| 认知科学发现 | 工程对应 |
|---|---|
| fuzzy-trace theory（gist/verbatim 双痕迹） | Claim 要义化思考 + Event 原文举证 |
| 选择性编码（海马体是编辑不是录音机） | AdmissionPolicy 准入规则 |
| 遗忘是主动信息治理（超忆症反例） | TTL/衰减/归档/显式遗忘 |
| 记忆再巩固 + 双重时间感 | supersede 链 + 双时间模型 |
| 睡眠巩固（海马体回放） | 后台 Worker 维护循环 |
| 源记忆错误（记得事、忘来源） | 全程 Evidence Link 证据链 |

^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

## 评测与边界

三层评测：3000+ 发布门禁（工程稳定性）→ 公开确定性基线（召回回归）→ 三套外部 Benchmark（横向坐标）+ 中文 E2E。LongMemEval（ICLR 2025，50 题）86%，与全上下文/Native RAG 只差 3 题和 2 题——证据/时间/生命周期治理未换来不可接受的信息损失；holdout50 实验验证"一次整理、多次复用"的结构化记忆成本优势 ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

适用边界：本地个人 Agent / 中文知识偏好记忆 / 需要来源追溯与被遗忘权场景。**不是**：多租户 RBAC 托管平台（namespace 仅逻辑分区）、默认图推理系统、全自动"类大脑"（Observation/Mental Model/Policy/Procedure 按真实使用证据逐步开放） ^[raw/articles/hl-mem-local-long-term-memory-governed-cognition-taobao-2026-09-11.md]

## 相关实体

- 与 [[entities/agent-memory-architecture-ruofei|若飞 Agent Memory 架构拆解]] 同主题不同视角：若飞拆解治理框架维度，本文是一手工程实现全貌
- [[entities/agent-memory-four-schools-comparison-2026-07-22|Agent 记忆四流派对比]] — HL-Mem 自定位：完整记忆治理 × 证据时间可解释 × 运行时独立 × 中文可用，区别于 Mem0（轻量提取）/ Zep·Graphiti（时间知识图谱）/ Letta·MemGPT（融进 Runtime）
- [[entities/state-of-memory-in-agent-harness-mem0-2026|Mem0 与 Agent Harness 记忆现状]] — 文中直接对照的轻量提取式路线代表
- [[entities/admem-memory-policy-selective-memory-sjtu-tencent-2026|AdaMem 选择性记忆]] — 同样主张"不是存得越多越好"，学术验证记忆治理必要性
- [[entities/trust-infrastructure-ai-data-retrieval-taobao-2026|AI 取数信任基础设施]] — 同信源（大淘宝技术）姊妹实践：取数侧的"确定性规则优先 + 门禁"与记忆侧同构
- [[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark|美团 VitaBench 2.0]] — 长期动态 Agent 评测基准生态
- [[entities/agent-memory-main-contradiction-context-scheduling|Agent 记忆主要矛盾：上下文调度]] — 记忆治理在上下文经济中的位置
- [[entities/taobao-live-anchor-agent-harness-engineering-2026|淘宝直播数字人 Agent Harness]] — 同为淘天直播技术团队出品
