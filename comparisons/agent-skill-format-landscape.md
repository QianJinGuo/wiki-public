---
title: Agent 技能格式版图
created: 2026-09-05
updated: 2026-09-05
type: comparison
tags: [agent-skill, format, portability, comparison, skills]
confidence: 0.65
provenance_state: merged
---

# Agent 技能格式版图（Skill Format Landscape）

> 2026-09 外部系统透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-external-probe|涌现稿]]观点三）的落地：用本地实证对比各家的技能打包格式，回答一个问题——**技能格式在趋同还是分裂？**

## 本地实证（两份一手样本）

| 维度 | ECC（社区，135 skills） | Hermes（本地运行时，3669 快照） |
|------|--------------------------|--------------------------------|
| 打包结构 | `<skill>/SKILL.md` | `<category>/<skill>/SKILL.md`（分类目录） |
| 核心字段 | `name` + `description` | `name` + `description` |
| 治理字段 | `origin: ECC`（社区溯源） | `version` + `author` + `license` + `metadata[本地运行时路径已隐藏]`（生产治理） |
| 字段风格 | 极简 | 版本化 + 溯源 + 运行时元数据 |

**结论：核心已收敛，分歧全在治理层。** 两者都用 `<skill-dir>/SKILL.md` + YAML frontmatter（name/description）——与 Anthropic 官方技能格式一致；差异不在"技能怎么写"，在"技能怎么管"：Hermes 按生产软件的标准治理（版本/作者/许可证），ECC 按开源社区标准治理（origin 溯源）。**"怎么写"已事实标准化，"怎么管"没有。**

## 厂商侧：本地证据缺口（诚实标注）

Doubao 本地 `skills/` 目录为**空**（无格式样本），CodeBuddy 的 Claw 目录为应用数据——**字节/腾讯系技能格式在本库尚无一手证据**。上一轮涌现稿担心的"厂商锁死"目前无法证实也无法证伪，保持开放。采集路径：各厂商官方技能市场文档（非本地应用数据）。

## 判断

对 [[concepts/skill-engineering-principles|Skill 工程原则]]的适用范围：核心写法跨平台可移植（结构性好消息）；治理字段不可移植——技能从社区搬到生产运行时需要补治理层（version/license/metadata），这是迁移成本的主要构成。对 [[concepts/agent-identity-portability|身份可移植性]]的启示：技能格式趋同走的是"事实标准先行、治理后置"路线，身份工件大概率重演同一剧本。

## 参见

[[entities/everything-claude-code|ECC 探针存档]] · [[concepts/skill-engineering-principles|Skill 工程原则]] · [[entities/agent-skill-writing-guide|Agent Skill 写作指南]] · [[concepts/agent-identity-portability|身份可移植性]]
