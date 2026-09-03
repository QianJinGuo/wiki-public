---
title: "Wiki 中 60+ 概念骨架的补建路线图（0 inlink + <2KB 模板）"
created: 2026-06-11
updated: 2026-06-11
type: moc
tags: [wiki-maintenance, concept-page, todo, roadmap, knowledge-gap, pending, skeleton, inlink-debt]
sources: []
confidence: high
---

## 元数据

> 本页是 wiki 自审自纠类页面，**不是知识内容**。用于追踪 60+ 概念骨架的补建进度。
> 任何 ingest 流程在创建新 concept 时应**先查本表**避免重复。

---

## 现状统计（2026-06-11 审计）

| 指标 | 数量 |
|------|------|
| 概念总页 | 149 |
| **0 inlink 骨架**（占总数）| **68**（其中 7 个已 archive，3 个已 expand，58 仍待补建） |
| **< 2KB 骨架 + 0 inlink**（占总数）| 54（7 已 archive，**3 已 expand**，**51 仍 active**） |
| 高 inlink 概念（>10）| 30 |

**根因诊断**：60+ 骨架都是同一模板自动生成的：frontmatter + 一句"汇聚 N 个相关实体的核心洞察" + 5 个 `<!-- TODO -->` 占位 + 一份"相关实体"列表。**承诺"汇聚 N 个实体"但内容部分未撰写**。

---

## 已 expand 的 3 个 P0（2026-06-11 commit e9d738da）

| slug | 原 size | 现 size | 核心内容 |
|------|---------|---------|---------|
| ai-self-improvement-bootstrapping | 2.3KB | **8.5KB** | 4 类自举路径 + lossy 风险 + 4 路径工程实践 + 5 启示 |
| prompt-injection-defense | 2.2KB | **8.4KB** | 3 类注入 + 三层防御 + Lethal Trifecta + 5 启示 |
| agent-security-attack-defense | 2.3KB | **7.7KB** | 5 类攻击面 + 4 层防御 + 评估方法 + 6 主题漂移实体剔除 |

→ 不 archive，**substantive expansion**。从 TODO stub 升级为有 4-5 个结构化章节的真正 concept page。
→ inlink 0→1/2/2（新增 1 entity 引用 + 1 query 引用 / 2 query 引用 / 2 query 引用）

## 已 archive 的 7 个（<700B 完全 placeholder，2026-06-11）

| slug | 原 size | 备注 |
|------|---------|------|
| world-models-simulation | 633B | 仅 2 entities 引用，且无实质内容 |
| moe-architecture | 637B | 仅 3 entities 引用 |
| scaling-laws-emergent-abilities | 642B | 仅 2 entities 引用 |
| reasoning-width-scaling | 677B | 仅 2 entities 引用 |
| chunking-retrieval-strategies | 809B | 4 entities |
| agent-error-recovery | 868B | 4 entities |
| speculative-decoding | 986B | 6 entities，但内容空 |

→ 全部 `git mv` 到 `_archive/`，index.md 同步删除。**理由**：<1KB 完全无内容，archive 后不损失信息（related entity list 已经在 entity 端保留）。

---

## 待补建骨架 51 个（按 size 升序）

> 全部具有 `<!-- TODO: 补充核心定义段落 -->` 占位 + 实体列表但无概念内容。
> **优先级建议**：size 越大 + 引用 entity 越多 → 优先级越高（说明该主题 wiki 已积累足够材料可写）。

### P0 — 高优先级（>8 个相关实体，主题核心）

| size | slug | 引用 entities | 主题 |
|------|------|-------------|------|
| 1358 | rlhf-dpo-grpo-alignment | 9 | PPO→DPO→GRPO 三代对齐算法 |
| 1360 | rag-framework-comparison | 10 | LangChain vs LlamaIndex vs Haystack |
| 1409 | agent-planning-reasoning | 15 | Agent 任务分解/链式推理/反射 |
| 1436 | workflow-automation-ai | 15 | DAG 编排/条件分支/AI 增强 |
| 1453 | knowledge-graph-rag | 15 | 图检索/实体关系推理 |
| 1470 | knowledge-management-ai-systems | 15 | 个人知识库/结构化索引 |
| 1487 | open-source-llm-ecosystem | 15 | Llama/Mistral/Qwen/DeepSeek 演进 |
| 1493 | enterprise-ai-adoption-patterns | 15 | POC→生产/ROI 评估 |
| 1495 | agent-evaluation-benchmarks | 15 | SWE-bench/WebArena/GAIA |
| 1524 | mcp-protocol-ecosystem | 15 | 工具注册/安全沙箱/标准接口 |
| 1530 | evaluation-harness-design | 15 | 可复现评测/自动评分 |
| 1547 | computer-use-agent | 15 | GUI 操作/浏览器控制 |
| 1564 | synthetic-data-generation | 15 | Self-Instruct/Evol-Instruct |
| 1576 | cognitive-architecture-ai | 15 | 感知-推理-行动循环 |
| 1599 | agent-observability | 15 | Trace/Span 审计链 |
| 1642 | prompt-injection-defense | 8 | 注入攻击/防御 | ✅ DONE 2026-06-11 (e9d738da) |
| 1646 | ai-self-improvement-bootstrapping | 8 | AI 自提升的引导机制 | ✅ DONE 2026-06-11 (e9d738da) |
| 1661 | llm-benchmark-landscape | 12 | 评测基准全景 |
| 1742 | agent-security-attack-defense | 10 | Agent 安全攻防 | ✅ DONE 2026-06-11 (e9d738da) |

### P1 — 中优先级（4-7 个相关实体）

| size | slug | 引用 entities | 主题 |
|------|------|-------------|------|
| 1123 | legal-ai-compliance | 7 | 合同审查/法规检索 |
| 1160 | model-distillation-compression | 7 | 知识蒸馏/结构剪枝 |
| 1292 | medical-ai-applications | 6 | 诊断辅助/药物发现 |
| 1295 | reasoning-models | 9 | o1/o3/DeepSeek-R1 演进 |
| 1586 | gpu-optimization | 14 | CUDA/显存/批处理 |
| 1604 | ai-testing-qa-pipeline | 7 | AI 测试流水线 |
| 1604 | prompt-engineering-patterns | 8 | 提示工程模式 |
| 1605 | vector-search-embedding | 6 | 向量搜索/嵌入 |
| 1608 | openai-model-evolution | 5 | OpenAI 模型演进 |
| 1609 | search-retrieval-agentic | 7 | Agentic 搜索 |
| 1624 | database-ai-integration | 6 | 数据库 AI 集成 |
| 1625 | software-testing-ai | 6 | 软件测试 AI |
| 1637 | data-quality-ai-pipelines | 6 | 数据质量 AI 流水线 |
| 1665 | long-context-techniques | 10 | 长上下文技术 |
| 1683 | code-generation-evaluation | 9 | 代码生成评估 |
| 1690 | game-ai-agents | 5 | 游戏 AI Agent |
| 1693 | coding-agent-comparison | 8 | 编码 Agent 对比 |
| 1713 | video-generation-models | 6 | 视频生成模型 |
| 1718 | ai-code-review-automation | 6 | AI 代码审查自动化 |
| 1726 | continuous-integration-ai | 5 | 持续集成 AI |
| 1766 | vision-language-models | 6 | 视觉语言模型 |
| 1767 | ai-ux-design-patterns | 4 | AI UX 设计模式 |
| 1779 | model-inference-comparison | 5 | 模型推理对比 |
| 1782 | llm-security-red-teaming | 6 | LLM 安全红队 |
| 1783 | observability-monitoring-ai | 5 | AI 可观测监控 |
| 1793 | nvidia-gpu-ecosystem | 5 | NVIDIA GPU 生态 |
| 1814 | harness-gate-evaluation | 6 | Harness 评估门 |
| 1820 | diffusion-model-architecture | 5 | 扩散模型架构 |
| 1847 | robotics-embodied-ai | 5 | 机器人/具身 AI |
| 1859 | fine-tuning-techniques | 6 | 微调技术 |
| 1926 | cloud-ai-infrastructure-design | 6 | 云 AI 基础设施设计 |

### P2 — 低优先级（<3 个相关实体，最骨架）

无（<3 entities 的 7 个已 archive 到 P0 阶段）

---

## 补建流程（推荐）

每补一个概念页，建议按以下流程：

1. **读相关实体**（列表中已有 5-15 个 entity）
2. **提炼核心命题**（1-2 段）
3. **写关键维度**（3-5 个分维度对比表）
4. **引用 ≥3 个 entity 加 `^[raw/articles/...]`**（per wiki-pipeline 2026-05-22 规范）
5. **至少 2 个 outbound wikilink 到其他 concept/queries**
6. **跑 lint** 确认 EXCESS INFERRED < 30%
7. **更新本表**（删除该行 + 累计"已补建 N/54"）

---

## 关联事件

- **2026-06-11 archive 7 个最 trivial**（<1KB，0 inlink）：避免 wiki 完整性破坏
- **2026-06-11 修复 sibling 竞态**：1 个 sibling subagent 创建的 `entities/diffusiongemma` 引用了 `concepts/moe-architecture`（已 archive）→ patch 为 plain text

## 相关实体 / 查询

- [[queries/llm-wiki-evaluation|LLM Wiki 评测方法]]（评测一次 ingest 的质量）
- [[concepts/agent-memory-system-design|Agent 记忆系统设计]]（31 inlink，concept 范例）
- [[entities/reverse-audit-prompt-paradigm-codex-5-line-version|反向审计 Prompt 范式（用 5 行 prompt 定期扫描知识库漏点）

## 待关联概念

- [[concepts/ai-cost-optimization-framework|AI 成本优化框架]]
- Human-in-the-Loop AI
- [[concepts/alibaba-llm-wiki-enterprise-practice|阿里数据团队 LLM Wiki 企业实践：LLM 编译思维构建结构化知识资产]]
