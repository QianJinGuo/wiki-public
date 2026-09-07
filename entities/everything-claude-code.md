---
title: "Everything Claude Code（ECC）——社区 Harness 蒸馏集"
created: 2026-09-05
updated: 2026-09-07
type: entity
tags: [claude-code, harness-engineering, community, agents, skills, hooks, probe]
source_url: https://github.com/affaan-m/everything-claude-code
confidence: 0.6
provenance_state: extracted
status: probe-archive
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Everything Claude Code（ECC）——社区 Harness 蒸馏集

> 本地探针存档页（2026-09 外部系统透镜轮）：`~/projects/everything-claude-code`，公开源 [github.com/affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)。社区维护的 Claude Code 生产级配置蒸馏集——**30 个专业 agents、135 skills、60 commands、自动化 hook 工作流**（SOUL.md 自述），1756 个 md 文件。

## 摘要

Everything Claude Code（ECC）是一个把「Claude Code 生产级用法」系统化、工件化、可移植化的社区仓库。它不是简单的提示词合集，而是把 **harness 的四个层——身份、规范、执行、元认知——都以文件形式显式建模**的可复制配置体系：SOUL.md 定义共享身份，RULES/CLAUDE/AGENTS 等描述规范，agents/skills/commands/hooks 承载执行能力，EVALUATION.md + dashboard 形成自我评估回路。它的价值在于展示了一个成熟开发者的 harness 到底是什么样的「第二个大脑」，以及这套东西如何做到跨人、跨机、跨项目迁移。

## 核心要点

- **身份层（SOUL.md）**：自我描述为「ECC 共享身份、治理与技能的**可移植层**（gitagent surface）」——人格、身份、治理边界被当作独立工件版本化管理，而非散落在各处 prompt 里。
- **规范层（配置即代码）**：`RULES.md`/`CLAUDE.md`/`AGENTS.md`/`agent.yaml`/`manifests/`/`schemas/` 把项目行为约束写成可校验、可分发的文件，含 schema 校验与 manifest 分发。
- **执行层（30+135+60）**：`agents/`（30 个分工 agent）、`commands/`（60 命令）、`skills/`（135 技能）、`hooks/`（自动化钩子）、`mcp-configs/`。
- **元层（可观测与自省）**：`EVALUATION.md`（自评估）、`REPO-ASSESSMENT.md`（仓库体检）、`ecc_dashboard.py`（可观测）、`the-longform-guide.md`/`the-shortform-guide.md`/`the-security-guide.md`（三层文档）、`research/`。
- **规模信号**：1756 个 markdown 文件——这不是玩具，而是一个「把一个人多年 Claude Code 功力全部固化成代码」的真实样本。

## 深度分析

### 一、ECC 是「harness 的实体化标本」

[[concepts/harness-engineering-framework|Harness Engineering 框架]] 通常被抽象地讲成「模型之外的所有工程」。ECC 的价值在于它**把 harness 的各个维度都落成了具体的文件与目录**，让你能直接翻读一个成熟体系长什么样：

- **身份不是玄学，而是文件**：SOUL.md 的存在证明「agent 的人格与治理边界」可以被版本化、被 diff、被审计。这正好呼应 [[concepts/agent-identity-portability|Agent 身份可移植性]]——身份的载体化是可移植的前提。
- **配置即代码不是比喻**：schema 校验 + manifest 分发意味着行为约束像软件一样经过「编译检查」，坏配置在运行前就被拦住。
- **分工是显式的**：30 个专业 agent 说明「多 agent」不是把 prompt 塞进一个上下文，而是把职责切成可独立演进、可独立评估的单元。

### 二、元层是它区别于普通 dotfiles 的关键

绝大多数开发者的 Claude Code 配置只有「执行层」（skills/commands/rules）。ECC 额外拥有 **EVALUATION.md、REPO-ASSESSMENT.md、ecc_dashboard.py** 这三个元工件——它给自己的 harness 装上了「体检 + 仪表盘」。这带来一个关键洞察：**一个成熟的 AI 开发工作流，必须包含对自己的持续度量**，否则 skills/agents 会像无人维护的 API 一样悄然腐化。这与 [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析]] 中「自进化内外双路径」的思路同构。

### 三、与 OpenClaw ／ Hermes 的对照

- [[entities/openclaw-architecture-8-part-summary|OpenClaw 架构]] 重在一个 agent 的运行时扩展（能力即外设），ECC 重在**一组分工 agent + 一套治理规范的协作形态**——前者像操作系统，后者像组织架构图。
- ECC 的 30-agent 舰队形态抛出了实时的问题：**30 个分工 agent 共享一个 SOUL 还是各自携带？** 抢上下文、责任边界、冲突消解都没有见文档——这正是大多数「agent 舰队」实践的无人区（参见 [[entities/claude-code-dynamic-workflows-thariq-practical-patterns|Dynamic Workflows 实战]]）。

### 四、可移植性的双重含义

ECC 的「everything」同时指：① 把 Claude Code 的所有能力维度（身份/规范/执行/元）都蒸馏进一个仓库；② 这个仓库可以**整体克隆到任何机器 / 任何项目**，一次 `git clone` 就带走整套 harness。这就是「技能格式收敛」叙事在个人层面的落地样本——当身份、技能、规则都以标准文件存在，换工具/换机器的成本趋近于零。

## 实践启示

1. **用「四层」自检自己的 harness**：检查你当前的配置是否只有执行层（skills/rules），缺了身份层（SOUL）与元层（评估/度量）。缺元层 = 会长期带着腐化的配置而不自知。
2. **配置即代码 + schema 校验值得立即拷贝**：把行为约束写成带校验的文件，而不是口头约定——坏规则在运行前暴露，而不是让 agent 在项目里犯一次错。
3. **可移植性从「身份文件化」开始**：把人格/治理边界从散落 prompt 中抽出来做成一个可版本化的 SOUL.md，是实现「换机不换脑」的第一步。
4. **多 agent 一定要显式切分工**：若要在项目里引入多个 agent，参考 ECC 的做法把职责切成独立单元，而不是把多个角色塞进同一上下文。
5. **给 harness 配一个 dashboard**：哪怕是 `grep` 统计 skills 数量 + 定期重读 EVALUATION，也比「凭感觉维护配置」强得多——度量化是防止配置腐化的最低成本手段。

## 相关实体

探针碰撞结论见 [[drafts/wiki-emergent-viewpoints-2026-09-external-probe|外部系统透镜涌现稿]]；正方簇：[[concepts/harness-engineering-framework|Harness Engineering 框架]]；同类探针：[[entities/openclaw-architecture-8-part-summary|OpenClaw 架构]]；身份层呼应 [[concepts/agent-identity-portability|Agent 身份可移植性]]；分工与自省见 [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析]]、[[entities/claude-code-dynamic-workflows-thariq-practical-patterns|Dynamic Workflows 实战]]；Karpathy 规则谱系见 [[entities/karpathy-claude-md-rules|Karpathy CLAUDE.md]]、[[entities/claude-md-12-rules-mnilax|CLAUDE.md 12 条规则]]。
