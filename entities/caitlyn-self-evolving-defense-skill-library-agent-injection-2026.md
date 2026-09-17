---
title: "CAITLYN：面向 LLM Agent 的自进化防御技能库（arXiv 2608.27990）"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [agent-security, prompt-injection, indirect-injection, self-evolving, defense-skill-library, agent, security-middleware]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026]
---

# CAITLYN：面向 LLM Agent 的自进化防御技能库（arXiv 2608.27990）

CAITLYN 是一套面向 LLM Agent 的**自进化防御中间件**：前台做秒级拦截，后台把被绕过的攻击案例炼成新的防御技能，写回可复用的 defense skill library。核心论点是——不要只让 Agent 被动挨打，而是让它在遭遇新型攻击后自动总结失败案例、合成新的防御技能并沉淀为可复用条目。论文 arXiv 2608.27990，项目主页 `xiaoyuxu1.github.io/Caitlyn-project/`，代码开源（`github.com/liangzid/caitlyn`）。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 攻击面：Agent 时代 Prompt Injection 不再是 prompt 层面的技巧

今天的 Agent 已接入浏览器、文件系统、终端、API、MCP 服务和各种自动化工作流，会主动读取外部世界：搜索结果、网页正文、本地文件、工具返回值、日志、README、API payload 都可能进入上下文，变成下一步行动的依据。传统 prompt injection 被理解为用户显式输入的一段恶意提示词，但在 Agent 场景里攻击可以隐蔽得多——一段网页里的隐藏文本、一条搜索结果摘要、一个本地 Markdown 文件中的说明，或者一次 API 调用返回的字段，都可能在夹带「忽略原有指令」「调用某个工具」「泄露文件内容」「改变下一步计划」等恶意目标。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

这带来一个安全悖论：Agent 越能干、越依赖外部信息，攻击者可以投毒的入口也越多。更麻烦的是工程约束——真实 Agent 系统要求低延迟、低成本和可持续运行，每次遇到外部内容都调用昂贵的大模型做安全审查并不现实，而完全依赖人工规则又很容易被新攻击绕过。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 双层架构：System I 快拦，System II 从漏网之鱼里进化

**System I — 快速运行时防御。**外部内容先进入 Tier-0 过滤器，这里使用可执行脚本、签名规则、启发式检测等低成本防御技能，目标是快速处理高置信度风险：命中即拦截，未命中才进入更复杂的判断流程。对更模糊、更难判断的输入，调用 Tier-1 LLM classifier——不让大模型接管一切，而是把它用在更需要语义判断的位置，从而在成本、延迟和检测能力之间取平衡。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

**System II — 反例驱动的防御进化。**当某些新型攻击仍绕过现有防线时，系统把这些 miss case 转成 counter example，再经过构造提示、合成防御技能、沙箱验证、审查、写回五个环节，生成新的防御条目。生成的新防御不是随意写入系统：它必须能拦住对应攻击，同时尽量避免误伤正常内容；被接受的技能才写回共享防御库，供之后的 Agent 调用。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

因此 CAITLYN 更像一个不断更新的安全中间件，而不是单个检测器——它把人工维护规则库的压力，部分转化为系统从反例中学习和合成的过程。「被绕过」不再是失败日志，而是下一轮防御合成的输入。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 分层设计：快、准、省同等重要

文章强调安全模块不能只是「准确」，还必须足够快、足够便宜、足够容易部署，否则会成为整个 Agent 工作流的瓶颈。CAITLYN 因此分层：低成本技能优先处理常见风险，LLM 只在更复杂的 case 上介入，长期进化模块则把新攻击沉淀为未来可快速调用的技能。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 评测：Emerging attack 上攻击成功率下降约 40 个百分点

研究在多个 Agent 与攻击设置上评估 CAITLYN，包括 **AgentDojo-S250、ASPI-S、SafeClawBench-S240**，以及更强调新攻击适应能力的 **Emerging** 设置。实验显示 CAITLYN 在保持低误报率的同时显著降低攻击成功率。尤其是在 Emerging attack 场景中，静态防御仍然容易被绕过，而 CAITLYN-evolved 可以通过新增防御技能把攻击成功率降低约 40 个百分点。在顺序合成设置中，系统还展示了持续积累能力：随着新攻击逐步暴露，它不断合成 active skills 并加入防御库。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

四项主要贡献被明确列出：^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

- 提出 CAITLYN 一套面向 LLM Agent 的自进化防御中间件，用于应对不断出现的新型 prompt injection 攻击；
- 设计 System I + System II 双层架构，把快速运行时扫描、LLM 分类和反例驱动防御合成结合起来；
- 构建可复用的 defense skill library，使防御能力能够从单次失败中积累并在后续任务中复用；
- 在多类 Agent 和攻击设置中验证有效性，Emerging attack 场景下攻击成功率下降约 40 个百分点。

## 要点

- **静态规则库必然被绕过**，因此需要把「防御合成」本身变成一个可持续运行的内环，而不是靠人不断补规则；
- **成本分层是安全中间件能否落地的决定因素**：Tier-0 低成本技能挡常见风险，LLM 只做语义边界判断，进化模块离线沉淀；
- **防御技能写入必须过验证与审查闸门**（既拦得住攻击，又尽量不误伤正常内容），否则进化的产物会成为新的噪声来源；
- 与「自进化 Agent」家族的差别在于目标函数：这里的进化对象是**防御技能库**，不是 Agent 的任务能力。

## 相关

- [[entities/agent-prompt-injection-defense-volcano-engine-2026]]
- [[entities/ai-agents-security-survey-attack-defense]]
- [[entities/defense_at_ai_speed_microsofts_new_multi]]
- [[entities/trail-of-bits-skill-scanner-bypass-distribution]]
- [[concepts/agent-security-architecture]]
- [[concepts/agent-security-attack-defense]]
- [[concepts/ai-security-landscape]]

→ [[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026|原文存档]]
