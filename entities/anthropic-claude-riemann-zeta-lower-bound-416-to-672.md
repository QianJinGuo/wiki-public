---
title: "Claude 黎曼 zeta 零点比例下限突破（41.6% → 67.2%）"
created: 2026-08-11
updated: 2026-11-22
type: entity
tags: [anthropic, claude, mathematics, multi-agent, formal-verification, lean, ai-capability]
sources: [raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Claude 黎曼 zeta 零点比例下限突破（41.6% → 67.2%）

## 核心结果

Anthropic 未发布的研究版 Claude 在尝试黎曼猜想时，意外将黎曼 zeta 函数满足假设的零点比例下限从 **41.6% 提升到 67.2%**。Claude 未证明黎曼猜想本身，但该结果是 AI 数学能力进展的里程碑式例证。两位 Anthropic 数学家验证了论文，Brian Conrey 与 Dan Goldston 两位领域专家审查确认，Claude 还产出通过标准验证工具 comparator 的 Lean 形式化证明。^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md]

## 多智能体编排细节

- 在 Claude Code 中两个会话完成，消耗 **3100 万输出 token**^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md]
- 非数学家 Jarred Sumner 只负责提示与鼓励（"继续"、"相信你自己"），数学选择完全由模型自主
- 首轮 650 个想法全部失败；重试后协调 **~60 个 Claude 子代理**，运行 **2400 条 shell 命令**、编写数百个 Python 脚本
- 子代理相互评审、对已知 zeta 零点做数千次数值检查、下载 54 篇 arXiv 论文确认结果未被发现、从零独立重证
- 用户输入以鼓励消息为主——帮助 Claude 克服训练中学到的"开放数学问题难度"自我怀疑

## 技术路径

将 Baluyot/Goldston/Suriajaya/Turnage-Butterbaugh 系列工作与 Bombieri 2000 论文结合。Claude 构造 Weil 诱导二次型函数空间，利用线上/线下零点的正/负定子空间，写出二次型秩的不等式（一阶/二阶矩信息）。关键勇气步骤：同时处理整个空间的正/负定性质，允许二次型非对角。^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md]

## 意义

1. **数学能力实证**：AI 模型可延伸数学家的想法，产出可验证的新结果——尽管技术路线本身不被期待能证明黎曼猜想
2. **意外发现范式**：结果为原任务的意外副产品，Claude 最初对自己的发现持怀疑
3. **多智能体自验证闭环**：60 子代理互审 + 反例搜索 + 文献查重 + 独立重证 + Lean 形式化，构成完整的 AI 科研验证流水线

## 深度分析

### 意外发现：科研突破作为副产品

这次突破最重要的方法论特征是"目标—产出"的错位：Claude 被要求认真尝试的是黎曼猜想本身（一个悬赏百万美元、159 年无人攻克的开放问题），最终交付的却是一个相邻但可完成的下界改进结果。原任务的意义不在于被解决，而在于提供了探索的方向约束——模型沿着正确的技术脉络（zeta 函数矩问题）推进，在能力边界处自然停在了够得着的成果上。这提示一种新的研究编排思路：给智能体设定"不可能完成"的北极星目标，其探索过程中溢出的副产品可能比刻意设计的中间目标更有价值。值得注意的是 Claude 对自身发现表现出明显的怀疑，这与训练数据中"开放数学问题极难"的先验直接相关——模型的自我认知先验在此构成了真实的性能瓶颈，而非仅仅是谦虚的姿态。这种"给大目标、收副产品"的模式与 [[entities/formalizing-fermats-last-theorem-claude-lean-anthropic-2026|费马大定理 Lean 形式化]]、[[entities/poincare-conjecture-lean-formal-verification-ai-2026-09-28|庞加莱猜想形式化验证]] 一脉相承。^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md:14-16,34-36]

### 多智能体自验证闭环的分层解剖

复盘整个流程可以看到一条清晰的验证金字塔，每一层针对不同的失败模式。第一层是子代理间的相互评审，捕捉推理链条中的逻辑漏洞；第二层是对已知 zeta 零点的数千次数值检查，用实证数据排除计算错误；第三层是反例搜索，主动证伪自己的结论；第四层是下载 54 篇 arXiv 论文做文献查重，防止"重新发现已知结果"这类学术上最尴尬的失败；第五层是从零开始独立重证，消除首轮推导中路径依赖的隐性错误；最后是 Lean 形式化证明通过 comparator 验证，把自然语言证明翻译成机器可检验的命题。这种"互审 + 反例 + 查重 + 重证 + 形式化"的五重结构之所以有效，关键在于各层错误检测机制彼此正交——数值检查抓不住的概念错误会被独立重证抓住，而两者都漏掉的证明歧义最终由形式化兜底。整体思路与 [[concepts/verifier-driven-development|验证器驱动开发]] 及 [[concepts/subagent-spawning-pattern|子代理派生模式]] 的实践相呼应。^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md:20-26]

### 组合式创新而非全新理论

从技术要点看，Claude 的突破并非发明了全新的数学工具，而是完成了一次高质量的文献综合：将 Baluyot、Goldston、Suriajaya、Turnage-Butterbaugh 的系列工作与 Bombieri 2000 年的论文拼接成一条前人未走通的路线。具体构造上，Claude 在 Weil 诱导的二次型函数空间中，利用线上/线下零点对应的正定/负定子空间，写出关于二次型秩的不等式（涉及一阶与二阶矩信息）。真正的"勇气步骤"在于同时处理整个空间的正/负定性质、允许二次型含非对角项——这一步在技术上不复杂，但需要突破前人因复杂性而回避的处理方式。这印证了 AI 在数学中的近期优势区：跨论文的知识综合、不怕繁琐的符号推演、以及不受"大家都不这么做"的惯例约束。人类的阻碍往往是审美和直觉层面的保守，而模型恰好没有这类包袱。类似的"多智能体分工产出研究论文"的编排方式在 [[entities/anthropic-multi-agent-research-system|Anthropic 多智能体研究系统]] 中也有系统化描述，[[concepts/multi-agent-collaboration-patterns|多智能体协作模式]] 提供了同类抽象。^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md:28-30]

### 成本结构与探索经济学

规模数据值得单独审视：两个 Claude Code 会话、3100 万输出 token、约一天半的挂钟时间、2400 条 shell 命令、数百个 Python 脚本，以及首轮 650 个想法的全部失败。这意味着最终成果的真实成本包含了极高比例的探索性浪费——首轮失败率是 100%。若把 token 成本视为科研预算，这项工作的性价比取决于如何看待"650 次失败"：它们不是浪费，而是模型在排除错误方向、收敛到正确技术路线的必要过程。对比人类数学家以年计的研究周期，一天半的墙钟时间展示了并行子代理探索在时间维度上的压缩能力——但代价是 token 维度的支出放大，这是典型的"用算力换时间"的科研经济学。^[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672.md:20-26,34]

## 实践启示

1. **领域决策交给模型，人类只做心理支持**：Jarred Sumner 全程不介入数学选择，输入以"继续""相信你自己"这类鼓励为主。编排者无需成为领域专家，其真实职能是维持智能体的执行动机。
2. **把失败预算写进工作流**：首轮 650 个想法全部失败是常态而非异常。设计智能体研究流程时应预设高失败率的探索阶段，用重试而非放弃来消化它。
3. **验证必须分层且正交**：单一验证手段（哪怕是形式化）无法覆盖所有错误类型。互审、数值检查、反例搜索、文献查重、独立重证、形式化各司其职，应作为流水线的固定环节而非可选步骤。
4. **模型先验会自我设限，需要主动干预**：Claude 因训练中学到"开放问题很难"而对成果持怀疑态度。长程任务中，编排者应识别这类自我怀疑信号并用鼓励性输入对冲，否则智能体可能在成功边缘放弃。
5. **形式化是对外建立信任的锚点**：自然语言证明不足以说服领域专家（Conrey、Goldston），Lean 证明通过 comparator 才让外部审查成为可能。AI 产出数学结果时，形式化应作为交付标准而非加分项。
6. **文献综合是 AI 当前最可靠的数学能力区**：本次突破来自既有文献的重新组合而非新理论。让智能体系统性地拼接跨论文路线，比要求它发明新工具的期望回报更高。

## 相关

- [[entities/ai-assisted-cryptanalysis-anthropic-claude-mythos-hawk-aes|AI 辅助密码分析（Anthropic）]]
- [[entities/agent-formal-verification-ai-code|Agent 形式化验证]]
- [[entities/agent-orchestration-multi-agent-systems|多智能体编排]]
- [[entities/anthropic-12-mcp-production-patterns|Anthropic 12 MCP 生产模式]]

→ [[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672|原文存档]]
