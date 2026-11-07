---
title: "OpenAI塌房！Scaling law原作曝bug，万亿算力全白烧"
created: 2026-07-05
updated: 2026-11-07
type: entity
tags: [article, llm]
sources: [raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OpenAI塌房！Scaling law原作曝bug，万亿算力全白烧



**   ****新智元报道  **

##### **【新智元导读】** DeepMind研究员深夜爆料：OpenAI的Scaling Law原始论文竟有致命bug！全球AI白白烧掉万亿算力，GPT-3其实严重「虚胖」。

  


OpenAI误导了整个AI圈好几年！

  


过去五年，整个AI行业都被Scaling Law推着往前冲。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


奥特曼坚信AGI的底气就来自这条曲线。

  


现在，有人站出来说：这条曲线，一开始就错了。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


不是事后诸葛。说这话的，是当年就在OpenAI做大模型优化的研究员**Diogo Almeida** 。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]

## 核心观点

> 本文通过article、llm视角，分析了的AI/ML技术动态。



**   ****新智元报道  **

##### **【新智元导读】** DeepMind研究员深夜爆料：OpenAI的Scaling Law原始论文竟有致命bug！全球AI白白烧掉万亿算力，GPT-3其实严重「虚胖」。

  


OpenAI误导了整个AI圈好几年！

  


过去五年，整个AI行业都被Scaling Law推着往前冲。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


奥特曼坚信AGI的底气就来自这条曲线。

  


现在，有人站出来说：这条曲线，一开始就错了。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


不是事后诸葛。说这话的，是当年就在OpenAI做大模型优化的研究员**Diogo Almeida** 。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


刚刚，他发出一篇博客，标题冷得发指——《Scaling Laws, Honestly》。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


开头一句直接把话说死：最初那版scaling law是错的，因为存在一个bug。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


传送门：https://www.completeskeptic.com/p/scaling-laws-honestly^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


DeepMind那位以扩散模型封神的**Sander Dieleman** ，转头就在推特上把它顶了上去，说这是一段有意思的LLM往事：^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


原始scaling law因为一个bug而错了，大概率害得业界在一堆「体量过大、训练不足」的模型上，白白烧掉了海量算力。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


  


一个bug，烧掉两年。

  


当bug被撕开，我们看到的，不仅是算力的黑洞，更是一条被语言本身重塑的、远比想象中更深刻的智能边界。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


  


**Scaling Law竟是LLM版「地心说」**^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


2020年，**OpenAI** 给出结论：在固定的算力预算下，你应该优先把模型做大，而不是拿更多数据去喂它。^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]


  


用公式说，最优参...^[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧.md]

## 技术洞察

本文的核心技术价值在于：
- ### 


**   ****新智元报道  **

##### **【新智元导读】** DeepMind研究员深夜爆料：OpenAI的Scaling Law原始论文竟有致命bug！全球AI...

→ [[raw/articles/openai塌房scaling-law原作曝bug万亿算力全白烧|原文存档]]

---
## 深度分析

### Bug 的真实成分：三重方法论缺陷，而非单一代码错误

从原文披露的细节看，这个「bug」并不是一行写错的代码，而是三层实验设计缺陷的叠加。第一层是「囚禁数据」：OpenAI 给所有模型——无论小模型还是大模型——喂了约 130B tokens 的相同数据量，小模型被喂饱甚至撑到，而真正需要海量数据填满容量的大模型在同一预算下严重「营养不良」。第二层是余弦学习率衰减：训练逼近预设终点时学习率被人为摁到零，loss 曲线随之走平，制造出「模型已饱和、再加数据无用」的假象——那不是模型的极限，而是学习率掐断了成长之路。第三层是论文中「结果基本不受学习率曲线影响」的表述：它在固定 token 上限的实验条件下技术上成立，却被当成了数据无限理想世界里的普适法则。连当年在 OpenAI 做优化的 Almeida 自己都没看出问题——那条学习率曲线看起来太像「精心设定」的了。

### 「万亿算力白烧」的认识论：标题级断言的证据结构

「万亿算力全白烧」是媒体标题，不是实测数字。原文的证据链是：Almeida 的博客断言最初那版 scaling law 因 bug 而错；Dieleman 转发时用的是「大概率害得业界……白白烧掉海量算力」的措辞——「大概率」本身就是证据等级信号。但方向上有独立佐证：Chinchilla 用不到 Gopher 一半的体量、四倍多的数据，在同等算力预算下全面反超，直接印证了「体量过大、训练不足」的系统性错配。合理的读法是：错配真实存在且被对照实验证实，「万亿」量级则是修辞推算而非审计结论；GPT-3「虚胖」的判断与 Chinchilla 之后的行业共识一致，可信度较高。

### 修正链的递归性：纠正者自身也带 bug

这条线的认识论教训比 bug 本身更深：2024 年 Besiroglu 等人重跑 Chinchilla 原文的数据点，发现其拟合里也藏着 bug——优化器里 loss 尺度设得过高，Huber 损失被按样本求平均而非求和，导致拟合过早终止。纠正 bug 的论文自己带着另一个 bug。Scaling Law 从来不是牛顿三定律那种铁打的物理规律，只是一条经验拟合出来的曲线；挂在嘴边的「第一性原理」在这里站不住。对任何 scaling 结论，可信度应挂在「数据点与拟合过程已被独立重跑」上，而不是挂在提出者的权威上。

### 语言偏差维度：你测的可能是「英语有多费算力」

Wasserman 指出了一个更底层的盲点：即便公式修正了，目前的 Scaling Law 也只是「英语 Scaling Law」。同架构同算力下，法语模型达到某种语法能力的效率比英语高出 50 到 100 倍——英语形态贫乏、过度依赖分布规律，需要模型在海量数据里猜词义；而法语、中文这类形态丰富或结构严密的语言，词汇本身就携带大量明确信息。这意味着现行所有算力配比方案，都建立在对数据最饥渴、效率最低的语言之上；以为在探索通用智能的「物理定律」，实际上可能只是在测量英语这门语言的浪费系数。

## 实践启示

1. 复现或引用 scaling 结论前先审实验协议：固定 token 预算 + 余弦衰减到零的组合会人为制造饱和假象，任何「再加数据也没用」的结论都要先检查学习率是不是被掐断了。
2. 对「白烧 X 万亿」类标题做两层拆解：机制是否有独立对照实验（Chinchilla 反超 Gopher 是硬证据），量级是否实测——「大概率」式措辞是证据等级的直接信号。
3. 采信经验拟合曲线前先问拟合器：scaling 结论的可信度应系于「数据点和拟合已被第三方重跑」，而非作者或机构的权威；修正版本身也可能带新 bug。
4. 做算力预算与采购决策时把语言/数据效率纳入变量：形态贫乏的语言更吃数据，跨语言场景照搬英语配比会系统性放大成本。
5. 组织层面：「当年在场」的内部专家复核并不足够——Almeida 自己承认当年也没看出 bug；关键结论的验证路径必须是外部独立重跑。
6. 认知策略上，把「权威结论 + 精心设计的曲线」视为待验证假设而非事实，警惕从「局部条件下为真」跳到「普适规律」的论证滑移。

延伸阅读：[[concepts/scaling-laws]] · [[entities/posts-2026-06-24-scaling-laws]] · [[entities/解构scaling-law优化架构数据的三重奏]] · [[entities/model-size-scaling-in-2023-2031]]

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

