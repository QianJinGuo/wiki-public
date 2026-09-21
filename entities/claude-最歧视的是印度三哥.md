---

title: "Claude 最歧视的，是印度三哥"
type: entity
created: 2026-07-04
updated: 2026-09-21
tags: [wechat, ai]
rating: v8c7
sources:
  - raw/articles/claude-最歧视的是印度三哥
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Claude 最歧视的，是印度三哥

**来源**: AGI Hunt

**发布日期**: 2026-04-29^[raw/articles/claude-最歧视的是印度三哥.md]


**原文链接**: https://mp.weixin.qq.com/s/_X_iaH_O__XMjpP4D-MRUg ^[raw/articles/claude-最歧视的是印度三哥.md]

---

你知道吗：同样一段话，用印地语发给 Claude，token 消耗是英语的 3.24 倍 。^[raw/articles/claude-最歧视的是印度三哥.md]


这个数字来自 AI 研究员 Aran Komatsuzaki 昨天做的一个实验。^[raw/articles/claude-最歧视的是印度三哥.md]


他把 Rich Sutton 那篇著名的《The Bitter Lesson》（苦涩的教训）翻译成了 7 种语言，然后分别丢进 OpenAI 和 Anthropic 的 tokenizer 里数 token。 ^[raw/articles/claude-最歧视的是印度三哥.md]

收费站

结果发现……Claude 的 tokenizer 对非英语用户，简直像在对外地人收过路费。^[raw/articles/claude-最歧视的是印度三哥.md]


01

## 测试方法

方法很简单。

Sutton 的《The Bitter Lesson》是 AI 领域一篇经典短文，原文是英文，长度适中，内容固定，很适合拿来做跨语言对比的基准。 ^[raw/articles/claude-最歧视的是印度三哥.md]

Aran 把这篇文章翻译成了印地语、阿拉伯语、中文、俄语、法语、西班牙语，然后分别粘贴到 OpenAI 和 Claude 的 token 计数器里。 ^[raw/articles/claude-最歧视的是印度三哥.md]

以 OpenAI 英文原文的 token 数为基准（1.00×），看看其他语言「膨胀」了多少。^[raw/articles/claude-最歧视的是印度三哥.md]


结果如下：

token 开销对比

蓝色是 OpenAI，橙色是 Anthropic。^[raw/articles/claude-最歧视的是印度三哥.md]


英语：OpenAI 1.00×，Anthropic 1.04×，几乎一样。^[raw/articles/claude-最歧视的是印度三哥.md]


西班牙语：OpenAI 1.18×，Anthropic 1.62×，Claude 开始拉开差距了。^[raw/articles/claude-最歧视的是印度三哥.md]


法语：OpenAI 1.30×，Anthropic 1.79×。^[raw/articles/claude-最歧视的是印度三哥.md]


中文：OpenAI 1.15×，Anthropic 1.71×。^[raw/articles/claude-最歧视的是印度三哥.md]


到了阿拉伯语：OpenAI 1.31×，Anthropic 2.86×。^[raw/articles/claude-最歧视的是印度三哥.md]


而印地语，OpenAI 是 1.37×，Anthropic 直接飙到了 3.24×。^[raw/articles/claude-最歧视的是印度三哥.md]


换句话说，一个印度用户用印地语和 Claude 聊天，同样的内容，要比英语用户多花 3 倍多的 token。 ^[raw/articles/claude-最歧视的是印度三哥.md]

02

## 不只是贵

拥有 20x Max 的你可能会想，多花点 token 也就是多花点钱嘛，反正我有的是不限量套餐。^[raw/articles/claude-最歧视的是印度三哥.md]


还真不是这么简单。

token 数量膨胀带来的连锁反应，远不止账单上的数字（并且套餐也有限额啊！）。价格只是第一刀，更为要命的是： 延迟 。 ^[raw/articles/claude-最歧视的是印度三哥.md]

3.24 倍的 token，意味着模型在开始生成回答之前，光是「读题」就要多花将近一倍的时间。

首 token 延迟（TTFT）直接被拖垮，用户的体验 会是 断崖式下降。^[raw/articles/claude-最歧视的是印度三哥.md]


三重打击

Aran 自己也算了一笔账：

“ 3 倍慢的解码速度，加上 3 倍频繁的 context 压缩，光想想就头疼。^[raw/articles/claude-最歧视的是印度三哥.md]


token 多了，输入处理慢了，输出也慢了，而且上下文窗口更容易被撑满，触发压缩的频率也更高。^[raw/articles/claude-最歧视的是印度三哥.md]


对于在生产环境跑 Claude 的印度开发者来说，这三重打击几乎是致命的：贵 3 倍、慢 3 倍、压缩 3 倍。 ^[raw/articles/claude-最歧视的是印度三哥.md]

03

## 扩大战场

Aran 觉得只比 OpenAI 和 Anthropic 两家还不够，于是又做了一轮更大范围的测试。 ^[raw/articles/claude-最歧视的是印度三哥.md]

这次他把模型扩展到了 6 家：OpenAI、Gemini 3.1、Qwen3.6、DeepSeek V4、Kimi K2.6、Anthropic。语言也增加到了 10 种，加入了日语、韩语、德语。 ^[raw/articles/claude-最歧视的是印度三哥.md]

结果

^[raw/articles/claude-最歧视的是印度三哥.md]

→ [[raw/articles/claude-最歧视的是印度三哥|原文存档]]

---
## 深度分析

### BPE 的词汇表，本质是市场份额的地图

主流 tokenizer 的训练方法叫 BPE（Byte Pair Encoding）：从单个字节起步，反复挑出训练语料里最高频的相邻 token 对合并成新 token，直到词汇表长到十万量级。这套机制里没有任何语言学意义上的公平性约束：谁的文本在语料里出现得多，谁的常用组合就先被合并成大 token。英文与代码占了大头，中文分到一块（市场大），印地语、阿拉伯语分到的份额就少得可怜，常用词只能退回按字节拆。“谁的市场大，谁的 token 就便宜”——单一的按量费率表看上去一视同仁，实际上是一张隐形的市场地图，这与 [[entities/anthropic_cache_tokenomics|Anthropic 缓存 tokenomics]] 里的单价精算叠加后，对非英语用户的实际成本弹性完全不同。%s

### 先天不平等：UTF-8 里天城文每个字符占 3 个字节

在 tokenizer 学会合并之前，还有一层更底层的编码差距：UTF-8 里英文字母占 1 个字节，印地语的天城文（Devanagari）每个字符占 3 个字节。也就是说，在任何合并规则生效之前，印地语的起点已经是英文的 3 倍。所以 3.24× 不是某次训练失误的偶然，而是「先天编码劣势 × 后天语料忽视」的乘积：编码层的劣势对所有模型一样，拉开差距的是厂商给多语言的语料权重。日语、韩语同样厄运难逃，“非英语”不是一个整体。%s

### 成本只是第一刀：TTFT、上下文压缩与套餐限额

3.24 倍的 token 意味着模型「读题」阶段就多花接近一倍时间，首 token 延迟（TTFT）被拖垮；输出解码同样更慢；上下文窗口更早撑满，触发压缩的频率翻倍。Aran 自己的说法是「3 倍慢的解码速度，加上 3 倍频繁的 context 压缩」——对生产环境里跑 Claude 的非英语开发者，这是贵 3 倍、慢 3 倍、压缩 3 倍的三重打击，而且即使不看账单，套餐限额也会更早耗尽。这与 [[entities/ai-agent-token-consumption-surpasses-humans-5x-2026|agent token 消耗超过人类 5 倍]] 是同一条线索：agent 本身已在放大词元开销，语言层膨胀把这个倍数又乘上一个系数。%s

### 跨语言评测的公平性：便宜的语言会被测得更好

同一个上下文窗口，在英文里能装下的内容量，在印地语里只能装下约三分之一。意味着同一个 agent，印地语会话会更早触发压缩、更早丢失中间对话细节，表现为「同一个产品在非英语用户手里变笨了」；而测评一般只报准确率、不报语言的 token 代价，于是“带着更少有效上下文参赛”的语言被系统性低估。对做 harness 与上下文预算的团队，这是个实打实的设计缺口：预算按 token 分配，「给用户多大的工作集」就成了语言的函数，而不是一个公平常量。这一点在 [[concepts/production-agent-engineering|生产级 Agent 工程]] 中通常被当细节忽略，对多语言产品却直接决定质量下限。%s

### 沉默的 Anthropic，与苦涩的反讽

原文没有记录 Anthropic 的任何回应。Aran 的归因是市场结构而非恶意：词汇表容量有限，像一本只有十万条目的词典，而重训 tokenizer 会冲击既有权重与缓存，代价极高，短期内不要指望因一篇推文而调整。这件事里最值得记住的反讽是选材：Aran 拿来做基准的《The Bitter Lesson》恰好论证着“人类手工设计的启发式方法最终都会输给算力”，而 tokenizer 正是这样一种启发式压缩方法——它的「高效」对某些语言完全是单方面的。也许公平问题未必靠打补丁解决，而会随更暴力、更粗粒度的新架构一同消失；在那一天到来之前，印地语用户每说一句话，都在交 3 倍的税。%s

## 实践启示

1. **把 prompt 语言当成显式参数**：不要默认「用户说什么语言就用什么语言做内部推理」。在成本或延迟敏感场景，把系统提示与思考链固定在工程上最便宜的语言（通常是英文），只把最终输出切回用户语言。%s

2. **按 token 而不是字符做预算与监控**：多语言的成本差异在字符维度上完全不可见，3.24× 只在 tokenizer 里现形。上线前用各家 tokenizer 实测，把每条语言路径的 token 数写进容量与成本模型。%s

3. **给多语言路径分层设阈值**：context 压缩触发点、max_tokens、重试预算都应按语言分层，否则印地语会话会更早压缩、更早撞上套餐限额，表现为「同一个 agent 在非英语用户手里更笨」。%s

4. **把 token 膨胀算进单位经济与定价**：按 token 计价的 API 对非英语市场隐含 2–3 倍溢价（6 家模型平均，Anthropic 2.07× 最高），做新兴市场定价时要么分语言定价，要么先把 token 效率写进毛利模型。%s

5. **跨语言评测要报成本，不只报分数**：只报准确率会掩盖「更贵的语言拿了更少有效上下文”的事实，把每语言的 token 膨胀率与有效上下文预算一并披露，横向结论才可比。%s

6. **把 tokenizer 效率当成选型一等指标**：Gemini（1.22×）与 Qwen（1.23×）平均膨胀率低于 OpenAI（1.33×），中文在中国模型上甚至比英文更便宜（Kimi 0.81×、Qwen 0.85×、DeepSeek 0.87×）。参考 [[entities/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026|Claude Code token 成本对比]] 这类同口径横测。%s

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

