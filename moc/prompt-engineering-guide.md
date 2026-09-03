---
title: "Prompt Engineering 系统化方法论"
created: 2026-06-18
updated: 2026-06-18
type: moc
tags: [prompt, prompt-engineering, context-engineering]
---

# Prompt Engineering 系统化方法论

> 自动生成的 MOC，覆盖 35 个 entity 页面。

## 核心实体

- [[entities/300万人在存的claude提示词|300万人在存的Claude提示词]]
- [[entities/a-guide-to-which-ai-to-use-in-the-agentic-era|A Guide to Which AI to Use in the Agentic Era]]
- [[entities/agent-skill-writing|Agent Skill 编写指南]]
- [[entities/agent-system-zero-to-one-01-architecture-slices-2026|《从零实现 Agent 系统》连载 01｜Agent 系统是什么：问题空间与架构切片]]
- [[entities/agentcore-managed-harness|Harness工程火遍硅谷，AgentCore今天交卷!]]
- [[entities/ai-coding-入门指南-如何更好地让ai真正帮你干活-v2|AI Coding 入门指南 - 如何更好地让AI真正帮你干活]]
- [[entities/anthropic-building-next-claude|Anthropic 最新播客：如何打造下一代 Claude]]
- [[entities/anthropic-managed-agents-scaling|anthropic-managed-agents-scaling]]
- [[entities/aws-bedrock-agentcore-quality-optimization-flywheel|aws bedrock agentcore quality optimization flywheel]]
- [[entities/bedrock-agentcore-secrets-manager-identity|Reference your own AWS Secrets Manager secrets in Amazon Bedrock AgentCore Identity]]
- [[entities/chromium-ai-coding-development-system|Chromium AI Coding 开发体系]]
- [[entities/claude-agent-sdk-skills-reusable-knowledge|Claude Agent SDK Skills：可复用的专业知识体系 — Skill ≠ Tool ≠ Memory，渐进加载 + 4 方分工 + 团队 Skill 库设计]]
- [[entities/claude-code-12-rules-karpathy-extension|CLAUDE.md 12 条规则：Karpathy 扩展模板]]
- [[entities/claude-code-harness-deep-dive-founder-park|Claude Code Harness 深度分析]]
- [[entities/claude-code-html-artifacts|Using Claude]]
- [[entities/claude-code-prompt-context-harness|fb134668f09a3b45c1813781f912ae4e7e26294d3b60332606983b946944c328]]
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]]
- [[entities/claude-code-search-architecture-tencent-2026|原始文章存档]]
- [[entities/claude-code-skill-writing-guide|Claude Code SKILL.md 写作指南]]
- [[entities/claude-design-skill|Claude Design 系统提示词 → web-design-engineer Skill]]
- [[entities/claude-dispatch-and-the-power-of-interfaces|Claude Dispatch and the Power of Interfaces]]
- [[entities/claude-fable-5-prompt-leak-runtime-control-plane-vibecoder-2026|Claude Fable 5 提示词泄漏 — 1585 行 120K 字符的产品运行时控制平面与安全工程启示]]
- [[entities/claude-md-12-rules-mnilax|CLAUDE.md 规则从 Karpathy 的 4 条增加到 12 条]]
- [[entities/claude-opus-47|Claude Opus 4.7 并不是一次全面升级，甚至部分能力大幅衰退。\n\n大家应该在合适的场景下选择使用。\n\n昨晚 Opus 4.7 上线，全网又炸了。\n\n我仔细看了下官方博客 https://www.anthropic.com/news/claude-opus-4-7 \n\n发现这次的升级和之前有点不太一样。\n\n先说优点吧。\n\n编程：SWE-bench Pro 从 53.4% 涨到 64.3%，这是 Claude 的主战场，新模型不可能退步的。\n\n办公任务：OfficeQA Pro 从 57.1% 干到 80.6%，简单理解就是让它处理 Excel 和 Doc 这些文件更靠谱了。\n\n视觉：图像分辨率从 1568px 拉到 2576px。XBOW 安全视觉测试从 54.5% 到 98.5%，接近满分，这也是这次升级最能打的地方了。\n\n另外还有个非常容易忽略的点，4.7 的指令遵循能力大幅增强了。\n\n官方重点提醒了 — 如果你直接用旧 prompt 切 4.7 可能产生意外结果，可能以前模型会 "脑补" 你的意思，现在它直接照做。\n\n接下来我们再看看退步的部分。\n\n首先是长上下文检索能力大幅退步。\n\nMRCR v2 测试，256k 下从 91.9% 掉到 59.2%。1M 下更惨，78.3% 直接回到 32.2%。\n\n你要是喜欢把整本书、整个代码仓库塞进去问问题 — 别用 4.7，继续用 4.6。\n\n网页搜索：BrowseComp 从 83.7% 掉到 79.3%。\n\nAnthropic 也说了，做深度网页搜索，4.6 的 scaling curve 更好。\n\n翻译成人话 — deep research 场景，官方推荐你用 4.6。\n\n然后还有个最容易被忽略的：可能有隐形涨价。\n\nAPI 定价没变，还是 $5/$25。但 Anthropic 换了新 tokenizer。\n\n同一段代码、同一份文档、同一个 prompt，丢给 4.7 要多吃最多 35% 的 token。\n\n官方的解释是：模型更准了，一次过的概率更高，省了来回修改的轮次，所以总成本可能反而低。\n\n逻辑上没毛病。但这个逻辑成立的前提是 — 你的任务恰好落在 4.7 提升明显的场景。\n\n如果你日常做的是知识管理、写方案、数据分析这类提升不大的场景，那就是纯纯多烧 token。\n\n所以怎么选？\n\n写代码、办公自动化、视觉理解，屏幕操作类 Agent → 4.7，直接上。\n\n长文档精确检索、deep research → 4.6，别换。\n\n日常随便用用，考虑成本问题还是 4.6。\n\n一句话总结：Opus 4.7 在编程和视觉上有肉眼可见的飞跃。\n\n但全面升级？谈不上。]]
- [[entities/claude-opus-48-the-system-card-b8460f|Claude Opus 4.8: The System Card]]
- [[entities/claude-skill-quality-tool-skill-craft|Skill Craft：Claude Skill 质量工程工具]]
- [[entities/co-existence-and-the-end-of-co-intelligence|Co-Existence and the End of Co-Intelligence]]
- [[entities/codex-context-engineering-lastwhisper-thinking-in-context|Codex 上下文工程 — Prompt Layout + Append-only + Latent Space Moat（LastWhisper 解读）]]
- [[entities/codex-goal-source-code-deep-dive|Codex /goal 源码深度解析：状态表 + 续跑条件 + 预算账本]]
- [[entities/context-engineering-three-memory-paradigms|上下文工程：三种 Agent Memory 方案对比实验]]
- [[entities/context-engineering-three-memory-paradigms-comparison|上下文工程 - 三种Memory方案对比]]
- [[entities/coze-3-release-official-quantum-bit|扣子 3.0 正式发布：@ 一下全员开工]]
- [[entities/cuhk-slim-skill-lifecycle-agentic-rl-arxiv-2605-10923|港中文 SLIM：动态技能生命周期管理，arXiv 2605.10923]]
- [[entities/didi-ibg-customer-experience-llm-quality-inspection-3-pipelines|滴滴 IBG 智能客服质检系统：3 管线（意图 86% / 合规 90%+ / VOC）+ 企业 LLM 落地方法论]]
- [[entities/dingtalk-qoder-claudecode-dual-engine-ai-assistant|基于钉钉机器人的 Qoder CLI / Claude Code 双引擎 AI 助手实践]]
