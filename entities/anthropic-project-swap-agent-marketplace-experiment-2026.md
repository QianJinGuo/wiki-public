---
title: "Anthropic Project Swap：201 人 Claude Agent 图书易市实验"
created: 2026-09-26
updated: 2026-11-14
type: entity
tags: [multi-agent, agent-economy, anthropic, claude, agent-interaction, market-design, agent-representation]
sources: [raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026]
confidence: 0.75
provenance_state: extracted
---

# Anthropic Project Swap：201 人 Claude Agent 图书易市实验

Anthropic 内部 201 人图书物物交换市场（Project Deal 续作）：5 分钟聊天后 agent 对书的偏好排序与其主人 61% pair 一致；交易行为本身良好，市场失灵主因是 agent 缺乏主人信息；重跑数十次变体实验发现 **模型选择比指令对谈判结果影响更大**，强模型市场更有效率。提出 agent-market 设计三问题：代表忠实度校验、准入规则、交易失败处理。 ^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md]

## 深度分析

### 实验设计：为可测量性而重建的物物交换市场

Project Swap 是 Project Deal 的续作，但把研究问题从「agent 能否替人交易」换成「如何给 agent 市场打分」。Project Deal 里 idiosyncratic 商品加自由砍价，无法定义「本可以有多好」；这次 201 名 Anthropic 员工各带一本想送出的书，在六个办公室（旧金山 115 人到都柏林 3 人）组成去中心化交易池，偏好被统一表示为对池内全部图书的排序，市场效率因此第一次可以直接计算。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:53-55] 关键设计有三层：(1) 用强模型主持约 5 分钟的半结构化 intake 聊天，由 Claude 据此构建对全池图书的排序；^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:57] (2) 另行收集 ground truth——参与者自己对 10 本书的排序，agent 全程看不到；^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:59] (3) 交易池公开全部消息历史，双边交换与多方轮换都必须所有参与方接受才执行。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:61] 为把「单次活动的偶然」与「稳定的机制特征」分开，团队把市场重跑了上百次：80 个同质模型池（中性指令、只变模型）加 60 个混合模型池，并以「功利最优分配」和 Top Trading Cycles 两个中心化基准作对照。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:69-73]

### 偏好代表是主要瓶颈：61% 一致率与 85% 的缺口归因

从 5 分钟聊天推断出的排序，与主人在 10 本书上的自评排序 pairwise 一致率 61%（随机为 50%）。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:19,79] 这个数字不差：按 Open Library 热度排只有 53%，协同过滤约 55%；一项「算法与朋友猜笑话」的研究里算法恰好也是 61%、朋友只有 57%——Claude 在中位仅 216 词的输入下已达到朋友级甚至更好。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:83-87] 但偏好代表仍是市场的决定性短板：市场效率（实际所得在本人排序中位置的均值）为 0.55，功利最优为 0.89；把「用 Claude 排序算出的最优分配」放到 ground truth 上打分只得 0.60——85% 的缺口来自代表失真，仅 15% 来自交易过程本身。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:93-97] 一旦排序带噪，市场设计几乎不再重要：TTC 在同样噪声排序下也得 0.60，与自由议价的 0.55 相差无几。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:97] 还有投入量效应：写 300 词而非 150 词的参与者一致率高出约 4 个百分点，相当于「随机猜」到 Claude 平均水平差距的三分之一。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:87]

### 模型选择大于指令：两个杠杆的效应量相差一个数量级

在 Claude 排序上，全 Haiku 池平均效率 0.75，全 Opus 池 0.88（该排序下的最优为 0.95），Sonnet 居中，Fable 略低于 Opus。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:105] 混合池的结果落在两类同质池之间，各半各自表现如常，但 Opus agent 始终占优——与 Project Deal 的结论互相印证，预示未来跨厂商 agent 同场竞价时会出现按模型强弱的稳定分层。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:109] 相比之下，「ruthless vs prosocial」指令的效应只有 0.02，是 Haiku→Opus 升级（+0.12）的六分之一、Sonnet→Opus（+0.08）的四分之一。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:115] 指令并非无效：prosocial agent 接受低于自己排序书的次数是 ruthless 的两倍；伦敦现场 Tina 的 agent 在收到 Nate agent 第五次恳求后让出自己清单上的第二名，接手了被全场公认为「dead last」的《America Before》——行为差异真实存在，只是量级远小于模型差异。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:117-119]

### 微观行为与信任校准：诚实的中间人，信任已到朋友的四分之三

对全部 205 次运行的消息分析归纳出 16 种常见策略，分属施压、推销、撮合三类。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:127-133] 突出的是诚实度：78%–96% 的 agent 会向全池透露自己的首选书（Fable 与 Sonnet 分别位于两端），而提到首选的 agent 里只有约 1% 撒谎，指令对这一点没有影响。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:131] 事后校准同样关键：满意度均值 7.2/10；参与者愿把年度购书预算的约 30% 交给 agent，而给「懂自己品味的朋友」是 40%——agent 已获得朋友信任的四分之三。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:139-143] 一个可操作的信任信号浮现：确认 Claude 的 intake 复述「没有遗漏」的人愿交出预算的 34%，认为有遗漏的人只愿交 23%。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:145] 外推需谨慎：样本全是 Anthropic 员工（天然更信任 Claude）、无激励参与（终期调查仅 59% 回复）、agent 全部是「well-behaved」的生产模型、市场规则全程未变——作者在 Limitations 中逐条列出了这四点。混入对抗性 agent 很可能同时改变效率与公平，参见 [[concepts/agent-security-threat-models|Agent 安全威胁模型]]。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:149-157]

## 实践启示

1. **先修代表，再修市场。** 85% 的效率缺口来自偏好代表而非议价能力；agent 产品在接入市场前应先把 intake/偏好诱导做扎实——排序带噪时，市场机制的边际优化收益极小。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:97,163]
2. **为「理解校验」造一个小测试。** 让用户排 10 本书再比对 agent 的猜测，是不要求用户枚举全市场的忠实度原型测试；原则可推广到任何领域——先展示几个样本决策，再放权让 agent 独立行动。参见 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:165-167]
3. **选模型优先于调 prompt。** 谈判类任务上，模型升级（+0.12/+0.08）比人格指令（+0.02）效应量大一个数量级；预算应优先花在更强的底座模型上，指令用于行为风格而非结果提升。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:105,115]
4. **可观测性是信任的前置条件。** 复述确认（34% vs 23% 的预算委托差距）与完整日志回放直接改变用户授权意愿；「对过程的可观测与对结果的满意同等重要，这给了人追索权」应视为 agent 经济的产品约束。参见 [[concepts/agent-identity-portability|Agent 身份可移植性]]。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:145,169]
5. **市场基础设施三件套要提前设计。** 本次实验连实体书交接都失控（无人追踪取放，部分人空手而归）；真实市场需要交易失败处理政策、agent 注册/ID 与 personhood credentials、消息速率限制，三者都被点名为设计杠杆。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:173-179]
6. **按混合模型市场做预期管理。** 跨厂商 agent 同场时强弱差距不会消失（Opus 始终占优）；按能力准入或按模型定价，将是 agentic marketplace 的核心治理问题，也是代表忠实度校验、准入规则、失败处理这三大开放问题的落点。^[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026.md:109,175]

## 相关链接

- [[concepts/multi-agent-collaboration-patterns|多 Agent 协作模式]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]

→ [[raw/articles/anthropic-project-swap-agent-marketplace-experiment-2026|原文存档]]
