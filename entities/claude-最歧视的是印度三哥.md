---

title: "Claude 最歧视的，是印度三哥"
type: entity
created: 2026-07-04
updated: 2026-08-01
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
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

