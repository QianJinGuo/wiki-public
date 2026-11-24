---
title: "Opus 4.7 发布：相比 4.6 核心变化与 Claude Code 搭配最佳实践"
created: 2026-06-11
updated: 2026-11-20
type: entity
tags: [claude, opus, claude-code, model-release, coding-agent, computer-use, agent, llm]
sources: [raw/articles/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2]
provenance_state: raw-linked
review_value: 7
confidence: 0.8
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: Opus4.7较短版本; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---
# Opus 4.7 发布：相比 4.6 核心变化与 Claude Code 搭配最佳实践

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/claude-4-5-sonnet-opus-release-notes|Claude 4/5 Sonnet & Opus Release Notes]] — 发布时间线与能力
- [[entities/code-as-agent-harness-survey|Code as Agent Harness 综述]] — 102页综述
- [[entities/anthropic-llm-attck-navigator-cyber-operations|Anthropic LLM ATT&CK Navigator: AI-Enabled Cyber Operations]] — ARiES风险评分
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道|深入理解 Claude Code 源码中的 Agent Harness 构建之道]] — 16095字源码8步循环
- [[entities/claude-code-7-layer-memory-architecture|Claude Code 七层记忆架构]] — 七层防御金字塔
- [[entities/claude-code-search-architecture-tencent-2026|原始文章存档]] — ripgrep五层过滤
- [[entities/claude-code-and-what-comes-next|Claude Code and What Comes Next]] — 压缩/Skills/Subagents
- [[entities/gpt-54-is-a-big-step-for-codex|GPT 5.4 是 Codex 的一次大跨越：四维评估视角与 Agent 战争回归]] — 四维评估+Claude/GPT哲学分歧5698字全版
- [[entities/claude-perceived-degradation-anthropic-effort-model-explanation-2026|全网骂Claude变笨，Anthropic下场揭秘：坑你的不是模型]] — Model vs Effort框架

## 工程实践
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]] — 并发与延迟加载深析
- [[entities/claude-code-agent-teams-task-decomposition-ruofei|Claude Code Agent Teams 实战：怎么拆任务、控权限、收证据]] — 拆任务控权限
- [[entities/andrej-karpathy-claude-md-134k-stars-2026|最佳 Claude Code 配置：Andrej Karpathy 的 CLAUDE.md，134+k star了！]] — CLAUDE.md四规则解析
- [[entities/claude-md-12-rules-mnilax|CLAUDE.md 规则从 Karpathy 的 4 条增加到 12 条]] — 12规则rv9主版
- [[entities/claude-code-source-leak-lifecycle-analysis|CLAUDE.md]] — 8步生命周期10k
- [[entities/deepseek-code-harness|DeepSeek Code Harness]] — DSH 28k主版
- [[entities/codex-major-update-appshots-goal-xinzhiyuan|Codex 重磅升级：Appshots / Goal 毕业 / 锁屏远程操控]] — 五能力升级
- [[entities/claude-code-official-plugins-anthropic|Claude Code 官方插件系统 (claude-plugins-official)]] — 官方插件五件套
- [[entities/knowledge-work-plugins-shuge-anthropic-deep-source|knowledge-work-plugins拆解：Anthropic官方开源，4 种组件、3 级加载、2 层记忆，纯文件的 AI岗位插件集]] — 岗位级封装+三级披露+两层记忆7956字
- [[entities/读完-claude-code-和-openclaw-的-memory-源码我对agent记忆需要向量数据库这件事产生了怀疑|读完 Claude Code 和 OpenClaw 的 memory 源码，我对 Agent 记忆需要向量数据库产生怀疑]] — 6层markdown记忆设计
- [[entities/gsd-get-shit-done-context-management-tool|GSD 上下文管理工具：用 Plan 约束 Agent 行为边界]] — GSD四层上下文+4档退化12811字
- [[entities/claude-code-dynamic-workflows-thariq-practical-patterns|Claude Code Dynamic Workflows 实战模式与构建技巧]] — 3失败6模式11用例
- [[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 上下文工程 —— Anthropic 团队的工程实践]] — 上下文工程官方表述
- [[entities/claude-code-skills-workflow-encapsulation-costa-long|Skills：让 Claude 记住「怎么做」，告别重复教学]] — context:fork隔离

## 深度分析

### 交互范式：从结对编程到任务委派
Anthropic 官方把 Opus 4.7 在 Claude Code 中的推荐用法从"逐行带着走的结对程序员"改写为"可委派任务的能干工程师"。这背后的能力支撑是：模型在更少指令下就能推理定义不清的任务，找 bug 与代码审查显著变强，跨会话保持上下文更稳定。交互结构因此被反转——与其把模糊需求摊到多轮里逐步补充，不如在第一轮就给齐意图、约束、验收标准与相关文件位置；多轮交互模式下模型会在每个用户轮次后做更多推理，长会话连贯性更好但 token 消耗更高，打包首轮往往同时改善效率与质量。^[raw/articles/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2.md:81-87] 与之配套的是 auto mode：权限请求路由给基于模型的分类器判定是否安全，安全即自动批准，人不必守着跑，还能并行开多个 Claude 实例——这是介于"保姆式盯梢"与 --dangerously-skip-permissions 之间的中间态。

### 自适应思考取代固定思考预算
Opus 4.7 不再支持带固定 thinking budget 的 Extended Thinking，改为 adaptive thinking：每一步是否思考都是可选项，模型按上下文自行决定把 thinking tokens 投到最可能有帮助的地方，且更不容易过度思考。配套的 effort 体系中，新的 `xhigh` 被设为 Claude Code 默认值——官方认为它对大多数智能体编码任务最优：自主性与智能强于 high，又不像 max 那样在长智能体运行中 token 用量失控；max 收益递减且易过度思考，只对当前会话生效，其余等级是会延续的"粘性"设置。官方明确建议升级后重新实验各 effort 等级而非照搬旧配置，也允许在同一任务中途切换等级来控制用量。^[raw/articles/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2.md:88-96]

代价也要看清：官方在注意事项中承认，新分词器使同样输入的 token 消耗增加 1.0-1.35 倍，且高 effort 等级下模型思考更久——尤其在智能体多轮对话的后期，输出 token 量明显增多。也就是说 xhigh 默认值换来的自主性是用单位成本上升换的，成本敏感型工作流需要主动降档到 medium/low（对范围明确的小任务，4.7 在低档的表现仍优于同档的 4.6）。此外 4.7 的定位仍次于限量发布的 Claude Mythos Preview：Anthropic 把它作为网络安全防护措施的"试验场"，这些能力最终会反哺 Mythos 的大规模推广——选型时应对两个模型分档管理预期。

### 静默的默认行为变化：模型变得更"克制"
针对 4.6 调优的 prompt 或 harness 在 4.7 上可能失效，根源是三处默认行为改变：回答长度按任务复杂度校准（简单查询更短、开放式分析更长）；工具调用频率更低但单次推理更多；默认生成更少 subagent。三者方向一致——算力从"动作"转向"推理"。依赖积极搜索/读文件或跨文件并行 subagent 的工作流，必须在 prompt 里显式声明何时用、为什么用；声明语气与文风时，正向示例比否定式"不要这样写"更有效。^[raw/articles/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2.md:103-108]

### 验证闭环：从人工职责内化为智能体流程
Claude Code 负责人 Boris Cherny 把"为 Claude 提供验证工作的途径"称为 2-3 倍产出效率的秘诀，在 4.7 上权重更高。按任务类型选验证通道：后端让模型知道如何启动服务做端到端测试，前端用浏览器扩展让 Claude 直接控制浏览器，桌面应用走 Computer Use。他的个人工作流收敛成组合技能 /go：先做端到端自测，再跑 /simplify 简化重构，最后直接提 PR。这与 4.7 模型侧"提交结果前自行验证输出"的改进形成呼应——验证从人工监督职责内化为智能体流程的一部分，对长任务尤为关键：几小时后回看进度时才能确信代码真的跑通了。这一思路与 [[entities/gsd-get-shit-done-context-management-tool|GSD 上下文管理工具]] 用 Plan 约束行为边界的做法同源：把约束与验收写进流程，而不是靠人盯着。^[raw/articles/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2.md:65-75]

## 实践启示
1. **首轮给足上下文**：意图、约束、验收标准、相关文件位置一次说清，减少往返轮次——每多一轮用户输入都是推理开销。
2. **effort 默认 xhigh 即可**：覆盖绝大多数编码任务；max 留给评测模型上限或极度依赖智能且对成本不敏感的场景，并记住它仅当前会话生效。
3. **升级后重调预算与 prompt**：新分词器使同样内容 token 消耗增加 1.0-1.35 倍，高 effort 长会话后段思考更久；预期输出文风用正向示例显式声明。
4. **给模型自验证通道**：后端配 E2E 启动脚本，前端配浏览器控制，或把"自测 → 简化 → 提 PR"封装成类似 /go 的组合技能——这是最直接的效率杠杆。
5. **需要并行或主动工具调用时显式要求**：4.7 默认更少 subagent、更低工具调用频率，跨文件并行等场景要写明何时用、为什么用。
6. **用 auto mode + 通知/Recaps 支撑长任务**：自动模式把权限判定交给模型分类器，配合完成提示音、Recaps 回顾与专注模式，实现多实例并行而人不必守场。

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]

→ [[raw/articles/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2|原文存档]]
