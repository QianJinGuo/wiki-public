---
title: "面向复杂业务场景的智能分析 Skills 架构设计与演进实践"
created: 2026-07-17
updated: 2026-09-07
type: entity
tags: [skills-architecture, knowledge-management, skill-design, knowledge-layering, routing-layer, token-economy, knowledge-decay, eval-driven-update, alibaba, local-life, analysis-skills]
sources:
  - raw/articles/alibaba-complex-business-skills-architecture-evolution
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 面向复杂业务场景的智能分析 Skills 架构设计与演进实践

阿里技术钟雨洁复盘面向本地生活业务的分析类 Skill 架构设计，经历 V1→V2→V3 三次重构，是覆盖几十个行业、跨多分析方法的领域密集型 Skill 架构设计的真实工程案例。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

## 架构演进

- **V1（软件工程思维）**：K/A/E 三层知识池 + 接口契约。失败：知识碎片化导致 context 爆炸，过度工程化
- **V2（知识收拢）**：行业知识收拢到独立文件 + 三级路由层 + 按需加载。新问题：行业文件越写越胖
- **V3（按变更频率分层）**：瘦行业文件（稳定层）+ 主题文件（时效层），写入/读取粒度分离^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

## 核心创新

### 三级路由层
域路由 → 行业路由 → 问题分类（是什么/为什么/怎么做），用路由 token 换取大幅减少无效知识加载。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

### 知识按变更频率分层
稳定知识（低频：经营框架/核心公式/指标定义）与时效知识（高频：策略/竞争/事件）分开存储。行业文件瘦身超 60%。写入和读取的最优粒度不一样原则。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

### 方法层做减法
20+ 方法压缩至 9 个。合并同类项、建立路由优先级（异常检测→归因→趋势→预测）、后置触发机制。消解信号，减少选项不消除歧义。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

### 表达层做框架
20+ 模板压缩至 4 类输出框架（监控/诊断/预测/汇报）。「约束结构释放内容」——定义骨架让模型填充。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

## 知识生命周期

**评测驱动更新闭环**：Eval → Diagnose → Register（登记旧值）→ Review（修改+校验）→ Re-eval。知识更新是一个有登记、有校验、有复测的工程流程。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

**反馈自演进**：静默采集用户纠错/追问/重复等信号，先入候选区观察，晋升后固化。弱模型友好（规则实现）。^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

## 六条设计原则

1. **收拢优于碎片化**
2. **按变更频率分层**
3. **选项少、信号强**
4. **约束结构，释放内容**
5. **知识保鲜靠机制不靠人**
6. **Token 经济性是架构的硬约束**—每个设计决策要回答"消耗多少 context 窗口"^[raw/articles/alibaba-complex-business-skills-architecture-evolution.md]

→ [[raw/articles/alibaba-complex-business-skills-architecture-evolution|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

