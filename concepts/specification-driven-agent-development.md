---
title: "Specification-Driven Agent 开发"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, sdd, spec-driven, agent, methodology, kairos]
sources: [concepts/sdd-specification-driven-development-harness, concepts/kairos-claude-code-paradigm]
---

## 定义

Specification-Driven Agent 开发：在让 agent 写代码前，先用结构化 spec（输入/输出/约束/边界）形式化任务。spec 既给 agent 做目标函数、也给 verifier 做验收依据。和「直接 vibe coding」对比，SDD 在严肃项目里把成功率从 30% 提到 80%+。

## 核心范式

- **spec 是 agent 的 prompt**：把模糊需求转成可验证的形式化描述
- **spec 同时是 verifier**：测试用例直接从 spec 派生，自动校验产出
- **spec 可被 agent 共同迭代**：用户和 agent 一起完善 spec，再 spec 驱动实现
- **适用边界**：复杂任务、多人协作、长期维护项目；玩具脚本 SDD overkill

## 背景与提出

Specification-Driven Development（SDD）在传统软件工程中已有成熟实践——先写接口契约和测试用例，再写实现。但当开发主体从人变成 AI agent 时，SDD 的意义发生了质变：spec 不再只是「文档」，而是 agent 的「目标函数」——没有 spec，agent 的行为就是概率性的随机游走。^[concepts/sdd-specification-driven-development-harness] 这让 SDD 从「最佳实践」升级为「必要条件」。

## 范式细节

在 agent 开发中，SDD 的 spec 需要包含 5 个维度。输入契约：agent 接收什么格式的输入、约束是什么、边界条件有哪些。输出契约：agent 产出的格式、质量标准、必须包含/禁止包含的内容。行为约束：agent 不能做什么（安全边界）、遇到异常时怎么做（错误恢复）、什么时候必须停下来等人类指令。验收标准：如何判断一次 agent 运行是成功的——不是「看起来对」，而是有可自动执行的 verifier。性能预算：token 上限、时间上限、重试次数上限——没有预算约束的 agent 会无限制地探索。这五个维度组成 agent 的「运行时宪法」，写在 system prompt 或 skill 文件里，每次 turn 都会被加载。^[concepts/kairos-claude-code-paradigm]

## 局限与反对声音

SDD 最大的问题是「写 spec 的成本」——一个复杂任务的完整 spec 可能比实现本身还长。对于探索性任务（「帮我调研 X」），spec 几乎不可能预先写全——因为调研过程中会不断发现新方向。另一个反对意见是：spec 本身可能有 bug，而 spec 的 bug 比代码的 bug 更难发现——因为 spec 的验证只能靠人读，不能靠自动测试。实用策略是「增量 SDD」：先写最小 spec 覆盖核心行为，运行后发现边界问题再补 spec，逐步收敛。

## 现实案例

Hermes Agent 的 SKILL.md 就是 SDD 在 agent 框架中的实现：每个 skill 有 frontmatter（输入/输出/约束）和 body（操作步骤/禁忌/验证方式）。当 agent 加载 skill 时，它读到的就是一份 spec——告诉它该做什么、不该做什么、怎么验证做对了。Hermes 的 wiki-lint pre-commit gate 则是 spec 的验证器：如果 entity 不符合 SCHEMA.md 定义的 spec（缺 frontmatter、wikilink 失链、无 provenance），commit 会被拒绝。^[entities/hermes-agent]

## 现实案例

Specification-Driven Agent 开发在 2026 年的三个最佳实践场域。Hermes Agent 的 skill 体系：每个 skill 文件本质是一个 spec——YAML frontmatter 描述元信息、SKILL.md 描述行为契约、references/ 描述边界条件。用户调用 skill_view 时就在加载 spec，agent 必须按 spec 执行。这种「spec 即 skill」的设计让 SDD 在 Hermes 中是默认行为而非额外步骤。^[entities/hermes-agent] Claude Code 的 subagent 定义：每个 Task tool 调用都需要 name + description + system prompt 三要素，这三要素就是 agent 的 spec——缺一项 agent 行为就不可预测。^[entities/claude-code-core-internals] Karpathy 在 YC AI Startup School 演讲中明确推荐：每次让 agent 写代码前先写一个 markdown spec，spec 必须包含「输入是什么 / 输出格式 / 成功标准 / 边界条件」。这套方法论被总结为「Spec First, Code Second」。^[entities/karpathy-vibe-coding-to-agentic-engineering]

## 实践启示

SDD 在落地时最容易踩的坑是「spec 完美主义」——花 2 小时写完美 spec 再让 agent 写 30 分钟代码，发现 spec 里 80% 的边界条件 agent 根本不会遇到。实用策略是「最小 spec 优先」：先写 5 要素的最小版（输入/输出/成功标准/边界条件/约束），跑通 agent 后看实际行为，再补 spec 覆盖发现的边界。这个迭代过程比一次性完美 spec 节省 60% 时间。另一个实践要点是「spec 即 prompt」——把 spec 直接放在 system prompt 的开头而非单独维护文档，这样 spec 不会和 prompt 漂移。Hermes 的 skill 文件就是这种设计：SKILL.md 既是 skill 的 spec，也是 agent 加载的 prompt。spec 的版本控制也很关键——spec 改了必须 git commit + 通知所有依赖此 spec 的下游 agent。否则会出现「spec v2 已发布但 agent 还在用 v1」的静默漂移。对于探索性任务（调研/头脑风暴/创意），SDD 反而是反模式——spec 会过早限制探索方向，应该用 vibe-style 自然语言 prompt。

## 与相邻概念的区分

SDD 和「TDD（Test-Driven Development）」的关系：TDD 是 SDD 在测试维度的特例——TDD 的 spec 就是测试用例，SDD 的 spec 更广（包含输入契约、行为约束、性能预算等非测试维度）。SDD 和「Design Doc」的关系：Design Doc 是写给人读的（团队 review 用），SDD 的 spec 是写给 agent 执行的（machine-readable）。好的 SDD spec 应该两者兼顾——人能快速理解意图，agent 能解析为运行时约束。SDD 和「Prompt Engineering」的关系：Prompt 是模糊的需求表达（自然语言），Spec 是精确的需求表达（结构化）。两者不是替代关系而是分层关系——Prompt 是「意图层」，Spec 是「契约层」，Spec 之下才是「实现层」。SDD 的反模式是「spec = 详细 prompt」——把 prompt 写得很长但仍然是自然语言，这不是 spec 而是 verbose prompt。真正的 spec 必须包含机器可解析的部分（YAML / JSON schema / 伪代码），否则 agent 无法严格遵守。

## SDD 工具链现状

2026 年的 SDD 工具链还不成熟。最完整的实现是 Hermes Agent 的 skill 系统（SKILL.md 作为 spec）。次完整的是 Anthropic 的 Claude Projects（用户上传的文档作为 context spec 但非执行 spec）。最不完整的是 LangChain 的 prompt template（只是 prompt 不是 spec）。SDD 工具链的几个缺口：spec 编译器（自然语言 spec → machine-readable schema）、spec diff 工具（spec 改了哪些下游 agent 受影响）、spec verifier（spec 本身的一致性检查）、spec 自动生成（从历史 agent 行为反推 spec）。这些工具都还在早期，是未来 1-2 年的创业机会。SDD 在企业落地时最常见的反模式是「spec 文档和 agent prompt 两套维护」——spec 改了没同步到 prompt，导致 agent 还在按旧 prompt 跑。解决方案是把 spec 直接嵌入 prompt 文件（Hermes Agent 风格），避免两套维护。SDD 的 ROI 在小型项目（< 100 行代码）上不明显，但在大型项目（> 10000 行代码）上是 3-5× 效率提升——因为 spec 驱动的 verifier 可以并行验证多个 agent 的输出，而人工 review 必须串行。

## SDD 与 LLM-as-judge 的协同

SDD 的 spec 还可以作为 LLM-as-judge 的输入——让另一个 LLM 读 spec 然后判断 agent 输出是否符合 spec。这是「verifier as LLM」的最小化形式，比训练专用 verifier 模型成本低很多。Hermes Agent 的 wiki review 流程本质就是这个模式：spec = 入库标准（v×c ≥ 49 + wikilink 完整 + provenance 合法），LLM-as-judge = web-content-reviewer 自动评分。这种协同让 SDD 不只是文档规范，而是可执行的验证流水线。

## 在 wiki 中的关联

- [[concepts/sdd-specification-driven-development-harness|SDD harness]]
- [[concepts/kairos-claude-code-paradigm|Kairos Claude Code 范式]]
- [[concepts/agentic-engineering-paradigm|agentic engineering 范式]]
- [[concepts/verifier-driven-development|verifier-driven development]]
- [[concepts/ai-coding-agent-from-helloworld-to-production|coding agent 从 helloworld 到 production]]

## 进一步阅读

- [[concepts/sdd-specification-driven-development-harness]]
- [[concepts/kairos-claude-code-paradigm]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
