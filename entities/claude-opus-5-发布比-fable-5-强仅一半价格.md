---

title: "Claude Opus 5 发布，比 Fable 5 强，仅一半价格"
created: 2026-08-30
updated: 2026-11-17
type: entity
tags: ['context', 'agent', 'ai']
sources: [raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Claude Opus 5 发布，比 Fable 5 强，仅一半价格

## Claude Opus 5 发布，比 Fable 5 强，仅一半价格

刚刚，Claude Opus 5 终于发布了！

Opus 5 发布

我 Claude 里的默认模型也已经悄悄换成了 Opus 5：

default opus 5

官方给它的定位是：

> “  Opus 5 是一个周到且主动的模型，以一半的价格，提供接近 Fable 5 的前沿智能。

措辞上说的是「接近」，但从跑分表看就会发现其实在大多数项目上，Opus 5 已经把 Fable 5 反超了。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

01

##  半价前沿

API 定价是每百万 token 输入 5 美元、输出 25 美元，和上一代 Opus 4.8 完全一致，但只有 Fable 5（10 / 50 美元）的一半。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

我们知道，Fable 5 是 A 社 6 月 9 日发布的最强公开模型，也是史上最贵的通用模型；同一底座的 Mythos 5 则只向 Project Glasswing 的少数合作伙伴开放。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

所以这次的 Opus 5 就是：一个半价的新模型，多数跑分反而压过了自家最贵的旗舰……

可用性上，所有付费套餐和 Claude API 今天直接可用，模型 ID 是  ` claude-opus-5  ` 。Claude Max 的默认模型已经切了过去，Pro 用户能用到的最强模型也是它。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

上下文窗口是 1M token，另外还提供一个 Fast 模式，速度约为默认的 2.5 倍（价格也翻倍……）。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

02

##  反超 Fable 5

先看官方给的这张总表，十几个项目里，Opus 5 拿下了绝大多数：

Opus 5 基准总表

Agentic 终端编码（Frontier-Bench v0.1）拿到  ** 43.3%  ** ，Fable 5 是 33.7%，Opus 4.8 只有 21.1%，一代直接翻了一倍还多。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

知识工作（GDPval-AA v2）  ** 1861  ** ，同样压过 Fable 5 的 1747。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

Agentic 搜索（BrowseComp）90.8%，电脑操作（OSWorld 2.0）70.6%，业务流程（AutomationBench）26.0%，全都是第一。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

SWE-bench Verified 也刷到了  ** 96.0%  ** 。

不过在 DeepSWE 上 GPT-5.6 Sol 仍以 72.7% 领先，法律和健康两项还是 Fable 5 和 Mythos 5 更强，无工具版的 Humanity's Last Exam 上，Fable 5 也还以 56.5% 对 56.3% 守住了一丝的领先。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

而且这表里还有个小乌龙是，FrontierCode 一项Opus 5 的 53.4% 被涂成了领先的高亮色，可旁边 Fable 5 明明是 53.5%…… ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

（这图不会是让 Opus 5 自己做的吧）

03

##  30.2% 的 ARC-AGI-3

ARC-AGI-3 的成绩方面：

ARC-AGI-3 成本对比

要知道该基准测的是模型解决从没见过的新题的能力，靠背题是刷不上去的。Opus 5 拿到了  ** 30.2%  ** ，而第二名 GPT-5.6 Sol 只有 7.8%，Opus 4.8 更是只有 1.5%。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

04

##  更省 token

Anthropic 这次反复强调的另一个点是效率：在相近甚至更低的单任务成本下，跑赢其他模型。

Frontier-Bench 成本曲线

几组官方数据如下：

•  Frontier-Bench 上，成绩是 Opus 4.8 的两倍多，单任务成本反而更低

•  CursorBench 3.2 上，开最高 Effort 档时，距离 Fable 5 的峰值成绩不到 0.5%，单任务成本只有一半 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

•  OSWorld 2.0 上，只用 Fable 5 三分之一出头的成本，就超过了后者的最佳成绩

•  AutomationBench 上，相同成本下任务通过率约是第二名的 1.5 倍；哪怕开最低 Effort 档，完成的任务也比其他所有模型都多 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

（注：Effort 是可调节的思考强度档位，档位越高，模型想得越多，也越贵）

OSWorld 成本曲线

Anthropic 官方人员 Alex Albert 表示，团队在跨领域的 token 效率上投入了大量工作，而他自己在很多编码任务上，已经更愿意用 Opus 5 而不是 Fable 5 了。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

AutomationBench 成本曲线

研究员 Nathan Lambert 的解读则是，这是更快的迭代速度加上规模化 RL 的产物，Fable 5 那个体量，目前反而还没办法好好做 RL。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]

HLE 成本曲线  0

## 深度分析

### 半价不是让利，是竞争逼出来的定价

Fable 5 是史上最贵的通用模型，同一底座的 Mythos 5 只向少数合作伙伴开放；Opus 5 则把 10/50 美元的价位直接砍到 5/25，与上一代 Opus 4.8 持平。文章结尾那句「感谢 Kimi K3，Fable 终于不再是奢侈品了」点破了真实背景：K3 这类开放权重旗舰把前沿智能的价格锚拉了下来，Anthropic 是在用半价换份额，而不是单纯技术普惠。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:29-33,169]

### 反超是真的，但边界很清晰

总表上十几个项目多数领先，但失守点都在垂直纵深：DeepSWE 由 GPT-5.6 Sol 以 72.7% 领先，法律、健康两项仍是 Fable 5 与 Mythos 5 更强，无工具版 HLE 仅以 0.2 分险守。更值得玩味的是 FrontierCode 的乌龙——53.4% 被涂成领先高亮色，旁边 Fable 5 明明是 53.5%。厂商跑分表的「领先色」是营销产物，读表必须自己核数，这与 [[entities/model-evaluation-from-benchmark-worship-to-self-built-evals]] 的结论一致。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:55-59]

### ARC-AGI-3 的 30.2% 是全场含金量最高的数字

该基准专测模型解决从没见过的新题，背题刷分无效。Opus 5 拿到 30.2%，第二名 GPT-5.6 Sol 只有 7.8%，Opus 4.8 只有 1.5%——对上一代近 20 倍的跳变，意味着在全新任务结构上的泛化能力出现了数量级变化，与常规跑分的渐进提升是两种性质。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:69]

### 效率叙事有工程支撑，但 Medium 档怪曲线提醒别拉满

Nathan Lambert 把效率优势归因于更快的迭代速度加规模化 RL——Fable 5 体量太大反而做不好 RL，与 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] 的规模化路径相印证。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:81-97] 但 FrontierCode 上最好成绩出现在 medium 档（53.4%），继续加档反而掉到 48.0% 和 43.6%：想得多不等于答得对，用力过猛会扣分。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:103-107]

### 对齐进步是最容易被低估的部分

自动化行为审计失准分 2.30，低于 Opus 4.8 的 2.85 与 Mythos 5 的 2.81，欺骗比例全家最低；安全分类器预期比 Fable 5 少拦约 85%，不会再莫名其妙降级。护栏允许在源代码里找漏修漏，生物类请求从拦截改为转给 Opus 5，但长时间自主研究仍受限——开放是分领域、有边界的，底座见 [[concepts/ai-safety]]。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:147-163]

## 实践启示

1. **默认模型切换已经发生**：Claude Max 默认模型已是 Opus 5，Pro 能用的最强模型也是它，API 模型 ID 为 `claude-opus-5`。新任务没有理由再指定 Fable 5——成本只有一半。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:35-37]
2. **Effort 档位按任务标定，别默认拉满**：FrontierCode 最好成绩出现在 medium 档，最高档反而掉分；机制分析见 [[entities/codexclaude-code-推理-effort本质-就是往prompt里塞了一句话]]。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:103-107]
3. **大规模跑批优先 Opus 5**：OSWorld 2.0 用三分之一成本超过 Fable 5 最佳成绩，AutomationBench 同成本通过率约为第二名 1.5 倍——评测、批处理类高量任务的成本差会被量直接乘出来。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:85-95]
4. **读厂商跑分表先核数**：53.4% 高亮对 53.5% 的乌龙说明领先色不可信；重要选型结论先用自家任务集复核再定。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:57-59]
5. **更新安全与权限预期**：此前被安全分类器误拦、误降级的场景会大幅减少；源代码漏洞研究已放开，生物类请求会转给 Opus 5 处理，但长自主任务仍有明显限制。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:151-163]
6. **审计数字可作采购参考**：失准分 2.30、欺骗比例最低是可引用的量化锚点，做模型风险评估时直接对照 2.85 与 2.81。 ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md:145-151]

→ [[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格|原文存档]] ^[raw/articles/claude-opus-5-发布比-fable-5-强仅一半价格.md]