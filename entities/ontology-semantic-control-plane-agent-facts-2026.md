---
title: "本体论（Ontology）：Agent 事实系统的语义控制面"
created: 2026-09-01
updated: 2026-09-03
type: entity
tags: [ontology, agent-facts, semantic-control-plane, knowledge-engineering, rdf, shacl, owl, ruofei]
sources: [raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08]
provenance_state: extracted
confidence: 0.8
---

# 本体论（Ontology）：Agent 事实系统的语义控制面

> -> [[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md|原文存档]]
> 架构师（JiaGouX）/ 若飞

## 摘要

若飞将本体论定位为 Agent 事实系统的**语义控制面**——摆在事实层和动作层之间，约束不同 Agent 对同一条事实的解释。核心贡献：四层架构（证据层→事实层→语义控制面→动作层）、claim 生命周期状态机（candidate→validated→published→retracted）、关系字典五要素、OWL 推理 vs SHACL 校验 vs 策略权限三层区分、大模型在本体中的角色定位（候选生成者而非发布者）。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 核心问题：一条边不够

RDF 三元组（subject/predicate/object）在真实业务里藏着五个未回答的问题：两端是否同一稳定对象、谓词的业务含义、关系生效/失效时间、来源材料与复核状态、哪些查询和动作有权消费它。W3C RDF 1.2 候选推荐明确提醒：RDF 抽象数据模型本身无时间，时间/版本/来源需显式建模。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## Claim 生命周期

生产事实存为可追溯的主张（claim）：claim_id + subject_id + predicate + object_id + valid_from/to + source_ref + status + schema_version。

status 状态机：**candidate**（模型提出，可进入核对但不能直接作为依据）→ **validated**（实体消歧+关系校验+人工复核）→ **published**（可被查询和动作消费）→ **retracted**（来源有误，不悄悄覆盖旧记录）。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 四层架构

| 层 | 负责什么 | 供应商案例 |
|---|---------|----------|
| 证据层 | 保存原文和上下文 | 合同、准入表、附件、审批记录 |
| 事实层 | 保存候选与已发布主张 | 谁是收款主体、何时有效、来源 |
| **语义控制面** | 定义类型、关系、约束与推理边界 | has_payee 两端类型、必需证据、多值规则 |
| 动作层 | 处理权限、策略、审批与副作用 | 能否付款、是否转人工、审计撤回 |

RAG 服务证据层，知识图谱服务事实层，本体在语义控制面，动作层独立存在。也适用于研发系统（CMDB 关系不能全压成 related_to）。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 本体管四件事

1. **对象**：稳定 ID > 名称。公司改名时身份不漂移。实体合并保守——保留待解析实体比强行合并更安全
2. **关系**：关系字典写清 domain/range/含义/inverse/cardinality/temporal/evidence_required。谓词越含糊，查询和动作越容易越权
3. **约束**：主张能否进入已发布事实。SHACL shapes graph 校验 data graph。工具后选，约束先说清
4. **推理**：事实/规则/推论分开保存。is_a/part_of/owns/calls 的继承/传递/反向要有明确规则和测试^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 推理 ≠ 校验 ≠ 授权

三层解决不同问题：
- **OWL**：开放世界假设，回答"根据现有事实和公理还能推出什么"
- **SHACL/领域校验器**：回答"这批数据是否满足发布条件"
- **策略与权限系统**：回答"满足条件后谁能执行什么动作"

把推理器当校验器，或把数据校验当授权系统，会在边界处留洞。缺失事实先视为未知，动作流程再决定暂停/补证/拒绝。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 大模型的角色

大模型负责**候选生成**：识别实体与关系、归一建议、按 schema 输出结构化主张、整理冲突来源差异。GraphRAG 标准索引和 OntoGPT SPIRES 方法都是这个定位。

发布权、全局语义修改权、高风险动作执行权应分别授权。模型可以读/提议/解释冲突，但生产事实的发布不应由模型直接决定。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 落地路径：从一条链开工

先选一条输入/输出/责任人都明确的流程，做四样东西：
1. **实体清单**：5-10 核心类型 + 稳定 ID + 别名/来源/匹配方法/合并记录
2. **关系字典**：10-30 高频谓词 + domain/range/含义/inverse/cardinality/temporal/evidence_required
3. **主张账本**：候选/已发布/已撤回 + 查询支持 as_of
4. **发布测试**：同名不合并/端点错误拦截/时间查询/来源撤回/缺失证据返回待复核

五个接口：submit_claim / validate_claim / publish_fact / query_facts(as_of) / retract_fact。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 何时上图

四种压力同时出现时图数据库才有吸引力：实体散落大量来源需持续对齐；跨两跳以上查询；关系按来源/时间/版本追踪；多 Agent 共同消费更新。否则 RAG 或关系数据库足够。稳定 ID、关系字典、事实生命周期和动作边界不会消失，存储可以替换，语义债务很难靠换库还清。^[raw/articles/ontology-semantic-control-plane-agent-facts-ruofei-2026-08-08.md]

## 相关实体

- [[entities/palantir-foundry-closed-loop-ontology-open-source-mvp-2026|Palantir Foundry 闭环操作范式：Ontology 三层]]（不同维度：Palantir 产品架构 vs 本文的工程落地方法论）
- [[entities/rag-vector-knowledge-graph-ontology|向量库·知识图谱·本体论：RAG 知识系统演进]]（不同维度：三种形态演进 vs 本文的四层架构+claim 生命周期）
- [[entities/enterprise-ai-ontology-agent-knowledge-governance|企业 AI 的非技术困境：本体驱动 Agent 与知识治理]]（不同维度：圆桌讨论 vs 本文的架构级拆解）
