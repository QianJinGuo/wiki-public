---
title: "Opus 5.5 无人值守停工问题与迁移指南：end_turn 语义变化与 Agent Harness 适配"
created: 2026-09-28
updated: 2026-09-28
type: entity
tags: [agent, harness, anthropic, claude, opus, long-running, api-migration]
sources: [raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡]
confidence: 0.75
provenance_state: extracted
---

# Opus 5.5 无人值守停工问题与迁移指南：end_turn 语义变化与 Agent Harness 适配

## 核心问题：Agent 干一半就停，是 harness 的错不是模型的错

Opus 5.5 发布后，跑 Agent 的开发者普遍发现模型干着干着就停下来，不催不动。根因不是模型偷懒，而是 **harness 的 stop 条件判定过时**：Opus 5.5 在长任务中会主动汇报进度，某些汇报发出后模型停止调用工具，API 返回 `end_turn`（本意是「这一轮我说完了」）；而沿用旧逻辑的 Agent 程序把「模型不再调用工具」等价于「任务完成」，一份进度汇报就这样被当成交差信号。Anthropic 官方在提示词指南中明确：**纯文本的回合结束应该看作一份汇报，绝不能当作任务完成的凭证**。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## 官方归类的四种半路停工模式

1. **纸上谈兵**：写完总结宣布下一步，但没调用任何工具，下一步永远停在口头上。
2. **过分礼貌**：停下来问「如果您不介意，我接下来继续处理某某」，然后原地挂机等用户回复。
3. **假装请示**：列出一串需要拍板的决策项，但这些决策按其自己的说法并不影响继续干活。
4. **汇报强迫症**：觉得当前回合字数够多或刚完成小阶段，非得停下来做总结。

讽刺的是，「沟通更主动、总结更清楚」正是 Opus 5.5 的官方卖点——这些优等生习惯放进旧的无人值守 harness 里反倒成了停工诱因。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## 官方三招：把验收权从模型手里拿回来

- **任务清单**：大任务拆细项交待办工具维护，回合结束时若清单仍有未完成项且模型未解释被什么卡住，应用自动发消息点名续跑（官方示例：「你的任务清单还有未完成项：迁移剩下两个端点……继续做。如果哪项被卡住，说明卡在哪里。」）。
- **铁面验收员**：事先定义完成标准，每次回合结束用更小的模型对照检查，不达标就把原因作为下一条消息塞回去返工。
- **硬刹车**：同一任务自动续跑两三次仍卡在原地就强制停下交给人复查，避免 API 额度空转烧光。

配套系统提示词两头堵：明说不想要上述四种停法，同时说清什么时候才准停（用户真推不下去 / 触碰被保护的核心资源）。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## 迁移陷阱：四处硬性 400 错误 + 三个暗坑

**硬性拒绝（不改直接 400）**：
1. `thinking` 不能再 disabled 或手工指定 `budget_tokens`——要么不传要么设为 `adaptive` 由 effort 控制思考深度。
2. `tool_choice` 不能再 `any` 或强制指定工具——改用 `auto` 配合严格工具调用/结构化输出。
3. thinking 块绑定模型与上下文：2026-08-31 后创建的账户，中途改系统提示词/工具/历史消息后回放旧 thinking 块默认报错（只追加不改写不受影响）。
4. 旧版电脑操作工具下线：Claude API / Google Cloud 换 `computer_toolset_20260801`；Bedrock 上旧 `computer_20251124` 仍可用。

**不报错的暗坑**：
- 两次工具调用之间的进度文字从正文（text 块）挪进了 thinking 块（默认 `display: omitted` 返回为空），界面只显示正文时长任务一片安静——解法是 `display: updates`（仅进度摘要）或 `summarized`。
- `max_tokens` 管思考加正文总量，旧的关思考时代的上限现在可能不够，回答写到一半截断。
- 思考内容不返回也照样按输出 token 计费。

返回处理代码两处易漏：分清思考块与正文块类型；工具来回调用时思考记录要原封不动传回（删改/调序都会被拒）。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## effort 档位语义漂移：同样的 medium 不是当年的味道

`effort` 成为调控成本的唯一旋钮（五档 low→max）。官方称 medium 档可追平甚至超越 Opus 5 的 high 档，但**同档位下 Opus 5.5 每回合思考量比 Opus 5 大得多（尤其高档位）**——旧项目 high 档原样搬过来回合变长、输出 token 狂飙。实操建议：从 medium 起步实测，确需提智商再上调；想少琢磨直接降档，比提示词里喊「别想太多」管用。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## 附带方法论：消除 AI 味靠拉黑名单而非抽象形容词

前端生成的提示词技巧：写「避免通用 AI 感」只会从一套 AI 模板换到另一套；有效做法是直接列黑名单（不要奶油色/灰白背景、不要标题斜体强调、不要 01/02 章节编号、不要等宽字体标签、不要胶囊形按钮）。模型能听懂具体的「我不要什么」，听不懂抽象的「我要好看一点」。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## 洞察

核心论断：**Agent 能否把活干完，模型能力只占一半；另一半看你怎么定义「完成」、怎么保存上下文、怎么分配推理预算。** 这与 [[entities/agent-harness-architecture|Agent Harness 架构]] 的核心命题一致——stop-reason 判定属于 harness 的回合控制层，不能依赖「无工具调用 = 完成」这种单信号启发式。与 [[concepts/harness-long-running-task|长任务 Harness]] 的任务清单 + 验收员 + 熔断三件套模式互相印证。模型代际更替时，很多应用的脚手架还停在上一代：过去逼模型写推理步骤，现在劝它少想；过去催它汇报，现在它汇报太勤反而停工。^[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡.md]

## 关联

- [[entities/agent-harness-architecture|Agent Harness 架构]] — stop-reason 判定与回合控制
- [[concepts/harness-long-running-task|长任务 Harness]] — 任务清单/验收/熔断模式
- [[entities/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定|Opus 5 系统提示词与 Agent 规则]] — 同模型家族 Agent 行为规范
- [[entities/introducing-claude-opus-5-on-aws-anthropics-most-capable-opus-model|Opus 5 on AWS]] — 前代发布与平台接入

→ [[raw/articles/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡|原文存档]]
