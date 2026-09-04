---
title: "Skill 工程原则"
created: 2026-07-02
updated: 2026-08-01
type: concept
tags: [skill, agent-skill, harness-engineering, agent, methodology]
provenance_state: inferred
confidence: 0.7
---

# Skill 工程原则

Skill 是 Agent 的可复用能力单元——类似函数之于程序。Skill 工程是 Harness Engineering 在能力封装维度的具体实践。

## 核心规范

- **6 字段规范**：name, description, trigger, steps, inputs, outputs
- **3 级加载**：description（概览）→ steps（展开）→ references（深读）
- **5 步评估闭环**：编写 → 测试 → 评估 → 迭代 → 发布

## 版本管理五大原则

语义化版本 + PR 模板 + 评估对比 + 灰度发布 + 回滚机制。Skill 的演进比代码更需要纪律——因为 Skill 直接影响 Agent 行为。

## 关联

- [[entities/agent-skills-development-guide|Agent Skills 开发指南]]
- [[entities/skill-version-comparison-five-principles-winty|Skill 版本对比五大原则]]
- [[entities/skill-version-management-semantic-versioning-practices-winty|Skill 版本管理五大原则]]
- [[entities/harness-skill-engineering-alibaba-practice|阿里云 Skill 工程实践]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
