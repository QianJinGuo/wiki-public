---

title: "EMNLP 2026 | ClawBench实测8款Agent：真能完成日常在线任务吗？"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [ai, agent, harness, multimodal]
sources: [raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗]
confidence: 0.75
provenance_state: extracted
---

# EMNLP 2026 | ClawBench实测8款Agent：真能完成日常在线任务吗？


# EMNLP 2026 | ClawBench实测8款Agent：真能完成日常在线任务吗？

让你更懂AI的 2026-08-31 13:51 北京

真实网页一测全员翻车

目前的 AI Agent 能够协助写邮件、整理文档等任务，但它们真的能在真实的网站上可靠地完成日常在线工作流吗？ ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

现有的 Web Agent 基准测试通常在离线沙盒或静态页面中评估模型。

为了真正迈向下一代通用 AI 智能体，MMLU-Pro 的作者联合滑铁卢大学 TIGER Lab、UBC NAIL Group 等共同推出了面向真实网页任务的新评估框架 ClawBench。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

这是一个面向真实生产环境网页任务的新评估框架，涵盖 153 个日常任务、144 个真实平台、15 个生活类别，并通过“最终请求拦截”机制确保安全评测。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

在 8 款前沿模型的测试中，最强的 Claude Sonnet 4.6 也仅取得 33.3% 的任务成功率，GLM-5 以 24.2% 的成功率与单任务 $0.64 的成本展现出最优性价比，Qwen 3.5 则以最低的 token 消耗和工具调用量兼顾了效率与可用性。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

论文进一步通过五层轨迹分析，揭示当前 Agent 失败的根源并非“不够聪明”，而是反爬虫机制下的无效重试、最后一步的确认犹豫、以及与人类截然不同的交互行为模式等深层原因。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

ClawBench 测试评估框架为评估下一代 AI Agent 提供了一个直面真实世界未解决问题的测试平台，保留了现实世界网页环境的完整复杂性、动态特性和交互挑战，并确保在不产生现实副作用的情况下安全评估。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

ClawBench 不仅仅给出了面向真实网页后"谁家模型最强"的榜单，更重新校准了当前人类对 AI Agent 真实能力的预期。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

论文作者：

UBC、Vector Institute、UniPat AI、CMU、滑铁卢大学、上海交大、港科大、清华大学、曼彻斯特大学、复旦大学、里海大学、南京大学 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

GitHub地址：

<https://github.com/TIGER-AI-Lab/ClawBench>

项目主页：

<https://claw-bench.com/>

论文链接：

<https://arxiv.org/abs/2604.08523>

Hugging Face：

<https://huggingface.co/collections/TIGER-Lab/clawbench> ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

什么是 ClawBench？

ClawBench 是一个专门针对日常在线任务的评估框架，包含了人们在生活和工作中最常遇到的 153 个任务。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

  * 它覆盖了从购物、预约餐厅到求职申请等 15 个类别的 144 个不同平台。

  * 与传统的“只读”或“离线”测试不同，CLAWBENCH 完全在生产环境（真实的在线网页）中运行。

  * 它侧重于评估“写操作密集型（write-heavy）”操作，例如填写复杂的表单、进行账户更新等改变网页状态的行为。




ClawBench 如何在真实网站上安全地测试？

如果让 AI 在真实网站上买东西或者投简历，会不会造成不可挽回的后果？

针对此，CLAWBENCH 设计了“最终请求拦截（final-request interception）”机制：系统会记录并拦截会导致真实世界副作用（如扣款、提交订单）的最终 HTTP 提交请求。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

这意味着 AI 可以完整体验真实网站的复杂性、动态渲染和验证逻辑，但在最后一步会被安全拦截。

八大模型折戟，最高战绩 33%

研究团队在 ClawBench 上对 8 款前沿 AI 模型进行了集中测试，并以 ‘Agent-as-Judge’ 协议作为核心指标来判定任务成功率，得出的结果出乎意料。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

1\. 总体成功率层面，日常在线任务的完成难度远大于受控浏览器中的完成难度：

  * Claude Sonnet 4.6 表现最好，但也仅仅达到了 33.3% 的任务完成率。

  * Qwen 3.5 排名第二，成功率为 26.1%。

  * GLM-5 以 24.2% 的成功率紧随其后。

  * 而 GPT-5.4 的成功率仅为 6.5%。




2\. 从任务层面的饱和度来看，低成功率并非仅由少数极端困难的任务造成。评测数据显示，在参与测试的 8 款前沿模型中， ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

  * 153 个评估任务里有多达 68 个（占比 44.4%）全军覆没，未能被任何一款模型成功解决。

  * 仅有 1 项任务被 7 款模型成功处理，

  * 没有任何一项任务能够被全部 8 款模型同时攻克。




3\. 模型不同领域能力表现分布不均衡：现阶段 AI Agent 无法在各种日常应用领域间实现能力均匀迁移。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

当前的榜单排名并非由某种单一的通用能力所决定。

具体而言，虽然 Sonnet 4.6 占据了总榜榜首，并在日常（Daily）、学术（Academic）及社交（Social）类任务中展现出最强实力，但其在金融（Finance）与宠物（Pets）领域仅与 Qwen 3.5 战成平手。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

作为综合排名第二的模型，Qwen 3.5 在旅行（Travel）场景中与 Gemini 3 Flash 并列第一。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

与此同时，GLM-5 在工作（Work）类任务中表现最为优异，而 Claude Haiku 4.5 则在开发（Dev）领域拔得头筹。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

这种能力分布的不均衡充分表明，现阶段的 AI Agent 尚无法在各种日常应用领域间实现能力的均匀迁移。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

换言之，一款能够自如应对购物或社交工作流的模型，在处理求职申请、差旅预订或开发者工作流时，可能依然会遇到巨大瓶颈。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

4\. 成本和性能表现：

绿色阴影标出的为「高效区间」（SR ≥ 15% 且成本可控）。能挤进这个区间的模型并不多。更关键的是：花更多钱 ≠ 更高的成功率，二者并不存在简单的单调关系。 ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]

综合评测，GLM-5 性价比优势

→ [[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗|原文存档]] ^[raw/articles/emnlp-2026-clawbench实测8款agent真能完成日常在线任务吗.md]