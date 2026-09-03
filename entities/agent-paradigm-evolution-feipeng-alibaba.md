---

title: "Agent核心技术概念与范式发生了哪些演变以及背后的思考"
created: 2026-06-10
updated: 2026-08-29
tags: [agent, architecture, data, database, fine-tuning, knowledge-mgmt, llm, memory, mlops, prompt, rag, search, tool-use, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/agent-paradigm-evolution-feipeng-alibaba
---

# Agent核心技术概念与范式发生了哪些演变以及背后的思考


## 相关实体

- [[entities/agent-system-zero-to-one-01-architecture-slices-2026|《从零实现 agent 系统》连载 01｜agent 系统是什么：问题空间与架构切片]]
- [[entities/cola-dlm-byte-dance-continuous-latent-diffusion-language-model|cola dlm：字节跳动连续潜空间扩散语言模型]]
- [[entities/lesecretairedefernand-co-en-tech-explicit-vs-implicit-in-the-age-of-intelligence|explicit vs. implicit in the age of intelligences — le secré]]
- [[entities/review-agent-how-it-decides-what-to-save-winty|review agent：后台复盘 agent 如何判断什么值得保存]]
- [[entities/不用再学ai了生成结果包稳的agent来了|不用再学ai了！生成结果包稳的agent来了]]
→ [[raw/articles/agent-paradigm-evolution-feipeng-alibaba|原文存档]] ^[raw/articles/agent-paradigm-evolution-feipeng-alibaba.md]

- [[moc/mlops-training-inference|MOC]]
## 核心观点
1. # Agent核心技术概念与范式发生了哪些演变以及背后的思考
**作者：** 飞樰
**发布日期：** 2026年6月1日
梳理 Agent 技术从2023-2026年的四个阶段演进（被动ReAct→工作流→自主→自进化）及六大核心维度（Prompt/Planning/Memory/Tools/Workflow/Environment）的技术概念变化。 ^[raw/articles/agent-paradigm-evolution-feipeng-alibaba.md]
2. 强调四个阶段非替代关系而是并存互补。
3. 核心洞察：宏观架构"形"未变，内核已重构——从"人为适配模型"到"利用模型原生能力"，从"刚性约束"到"动态智能"。
4. （本文覆盖的4阶段+6维度Agent框架已由 entity [[entities/agent-evolution-four-stages-six-dimensions-aliyun|Agent 四阶段演化与六维度技术变化]] 完整收录。
5. ）
### Prompt：渐进式披露
System Prompt 从"单体大作文"到"System Prompt + 渐进式加载上下文文件"的解耦。 ^[raw/articles/agent-paradigm-evolution-feipeng-alibaba.md]

### 关联实体

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]

