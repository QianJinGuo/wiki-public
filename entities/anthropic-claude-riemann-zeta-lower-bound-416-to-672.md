---
title: "Claude 黎曼 zeta 零点比例下限突破（41.6% → 67.2%）"
created: 2026-08-11
updated: 2026-09-07
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

## 相关

- [[entities/ai-assisted-cryptanalysis-anthropic-claude-mythos-hawk-aes|AI 辅助密码分析（Anthropic）]]
- [[entities/agent-formal-verification-ai-code|Agent 形式化验证]]
- [[entities/agent-orchestration-multi-agent-systems|多智能体编排]]
- [[entities/anthropic-12-mcp-production-patterns|Anthropic 12 MCP 生产模式]]

→ [[raw/articles/anthropic-claude-riemann-zeta-lower-bound-416-to-672|原文存档]]
