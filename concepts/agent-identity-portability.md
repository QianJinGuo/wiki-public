---
title: Agent 身份可移植性
created: 2026-09-05
updated: 2026-09-06
type: concept
tags: [agent-identity, portability, harness, persona, governance, soul]
confidence: 0.65
provenance_state: inferred
---

# Agent 身份可移植性（Agent Identity Portability）

> 2026-09 外部系统透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-external-probe|涌现稿]]）识别的簇内缺失原语：harness 簇覆盖了上下文/工具/评测/记忆，唯独没有"**agent 身份作为独立、可移植的工件**"这一设计判断。

## 原语定义

把"这个 agent 是谁"（人格、价值观、语气、治理边界）从执行环境（system prompt、产品壳、具体模型）中**剥离成独立版本化工件**，使其可以跨工具、跨模型、跨产品迁移。判据：换掉底层模型或宿主产品时，身份定义零改动可用。

## 库内证据

**正面实例**：[[entities/everything-claude-code|ECC]] 的 `SOUL.md` 自述为"共享身份、治理与技能的**可移植层**（gitagent surface）"——身份不是 prompt 里的一段话，而是一个有独立生命周期、可分发、可继承的文件。与之配套的是规范层（RULES/manifests/schemas），说明身份在 ECC 里是被**治理**的（版本化、校验），不是被"调"出来的。

**弱佐证**：Doubao 的 `chats/` 与 `skills/` 分离存储（主体状态与能力分开打包）；CLAUDE.md/AGENTS.md 惯例（[[concepts/harness-engineering-framework|框架页]]工件分类）事实上是"项目级身份"，但它们只覆盖指令，不覆盖人格与治理边界。

**簇内缺口**：wiki 有 role/persona 的零散讨论（[[entities/role-confusion-github-io|Role Confusion]]从攻击侧讨论角色边界），但没有"身份层与执行层分离、身份可移植"的正面设计页。

## 开放问题

1. **身份与系统提示的边界**：身份工件和 system prompt 的分工线在哪？前者管"我是谁"，后者管"我怎么干活"——但两者混写是常态，分离的实操判据缺失。
2. **多 agent 组合下的身份**：30 个分工 agent（ECC）共享一个 SOUL 还是各自携带？ECC 的答案是共享层 + 差异化 agent 定义，但冲突消解机制未见文档。
3. **身份的攻击面**：可移植身份文件被注入/篡改时，治理边界跟着失守——与 [[concepts/channel-enumeration-criterion|通道枚举]]的交点，尚无分析。

## User Story 与真伪判定（2026-09-06 会话评审）

> 2026-09-06 会话对本原语做的 User Story 拆解与真伪评估，纯推演、无外部源（inferred，不加 citation）。

### User Story 三档

- **多工具重度用户（当下真实、人群小）**：同一身份需在 Claude Code / Hermes / Cursor 逐工具重写，换工具即"忘了我是谁"。痛点 = 重复劳动 + 漂移，仅 2+ 工具重度用户体感。
- **agent 舰队团队（当下最疼、最可能付费）**：数十个分工 agent 的人格与行为边界散落各自 prompt，无整体治理视图、无可审计清单。需要集中定义 + 版本化 + 下发；但本质接近配置管理，git + CLAUDE.md 已缓解八成。
- **持久化 agent 用户（尚未存在、在路上）**：agent 积累的默契与边界是多年资产，换平台应随人迁移（类比携号转网）。身份 = 持久化 agent 资产的便携层，本原语的最终归宿。

### 判定：真问题、假时机（非伪需求、亦非当期品类）

激励不对称论证：本原语的姊妹发现是技能格式正在**收敛**（一手核查 2026-09-06：Cursor 官方文档原生直读 `.claude/skills/`，Codex 与 Claude Code 为同一标准的两块牌子，CodeBuddy 亦为 SKILL.md 式，社区转换器项目已存在）——这同时构成对 [[drafts/wiki-emergent-viewpoints-2026-09-external-probe|外部探针稿]]"三家各自内化、趋于分裂"预判的反例（待该稿修正）。对比之下身份零收敛迹象，原因在激励：**技能是加法**（开放标准使平台生态变强，故自发收敛），**身份是护城河**（人格与个性化理解是留存率来源，平台无动机使其可迁移）。推论：身份可移植不会自然发生，只会经三路到来——用户以文件自绕（SOUL.md 社区实践）、组织内标准化、终局由规范/协议强制。

### 可判定判据（台账候选，未登记）

- **正方赢**：2027-12-31 前出现身份/人格可移植的开放标准尝试，或某主流平台主动支持身份导出。
- **反方赢**：届时主流平台反向加深人格/记忆的单生态锁定（个性化做得更深但仅限自家生态）。

符合 [[queries/prediction-ledger|预测对账台账]]登记标准（到期可判定、双侧预写证据口径），暂不登记——等该原语有实质素材轮再顺手入账。

### 操作结论

概念页保留作 discourse 卡位（"激励不对称"角度全网暂无透写者）；不建工具、不立项；作为断言三轴（trustwiki / 断言半衰期 / 预测台账）主线的叙事储备，待 agent 持久化（记忆、长寿命会话）成为大众话题时启用。

## 参见

[[entities/everything-claude-code|ECC 探针存档]] · [[concepts/harness-engineering-framework|Harness 框架]] · [[concepts/agent-role-specialization|Agent 角色专业化]] · [[comparisons/agent-skill-format-landscape|技能格式版图]]（姊妹迁移问题）
