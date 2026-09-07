---
title: "快手AI测试四阶进化实践 — Prompt→Multi-Agent→知识工程→Agentic自进化"
created: 2026-07-09
updated: 2026-07-09
type: entity
tags: [ai-testing, ui-testing, test-case-generation, agentic-workflow, knowledge-engineering, multi-agent, rag, badcase-driven, self-evolving, kuaishou, harness]
sources:
  - raw/articles/智能ui用例生成与执行的四阶进化实践
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 快手AI测试四阶进化实践 — Prompt→Multi-Agent→知识工程→Agentic自进化

快手研发Agent负责人苗星在 AiDD 大会分享的AI+测试演进路径，从Prompt工程到Agentic自进化的完整四阶实践。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

## 核心框架

### 四阶演进路径

| 阶段 | 方法 | 核心能力 | 生成率 | 采纳率 |
|------|------|----------|--------|--------|
| V1.0 | Prompt工程 | Few-shot + 场景化Prompt | 8% | 40% |
| V2.0 | Multi-Agent协作 | PRD解析→用例生成→用例评审Agent | 15% | 55% |
| V3.0 | 知识工程 | RAG+多模态+历史缺陷+业务私域知识 | 35% | 65% |
| V4.0 | Agentic自进化 | ReAct Loop+Review-Critique+BadCase自进化 | 70% | — |

### 关键洞察

- **测试效率瓶颈：** 不在执行自动化，而在用例生成、质量保证和规模复用^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]
- **生成质量上限由知识丰富度决定**，而非Prompt精妙程度^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]
- **BadCase系统化收集是自进化的唯一可靠燃料**^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

### 核心技术组件

1. **V1.0 Prompt工程：** 3-5个Few-shot样本 + 场景化Prompt模板 + 输出强约束。失败原因：长文档衰减(30K+ tokens遗忘)、业务知识缺失、过程黑盒^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]
2. **V2.0 Multi-Agent：** 复制QA工作流为三个Agent（PRD解析→用例生成→用例评审），AI负责生成，人负责Review^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]
3. **V3.0 知识工程：** 四种知识资产注入（文本测试物料、多模态物料、历史用例RAG、业务私域模板），历史缺陷覆盖率从12%→76%^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]
4. **V4.0 Agentic自进化：** 三层资产（170+ Skills、Memory、Knowledge图谱） + Review-Critique双级评审 + BadCase规则库自进化。场景规则模板更新从天级降到5分钟，维护成本下降99%^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

### 智能用例执行

测试意图驱动的第三条路（非纯脚本也非纯自由探索）：AI-Friendly用例（原子化+可观测预期+环境显性化）+ 执行引擎"四觉"能力（感知→决策→执行→反馈）^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

### 方法论建议

1. 从高价值低复杂度场景开始
2. 历史缺陷库 > 业务规则模板 > Prompt优化
3. 从V1就要设计反馈闭环
4. 明确人机分工边界^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

→ [[raw/articles/智能ui用例生成与执行的四阶进化实践|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

