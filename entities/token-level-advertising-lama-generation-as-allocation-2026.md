---
title: "Token-Level Advertising：LAMA 把拍卖写进自回归生成（人大高瓴 × 斯坦福）"
created: 2026-09-21
updated: 2026-09-21
type: entity
tags: [llm, agent, mechanism-design, decoding, multi-agent, economics, advertising]
sources: [raw/articles/token-level-advertising-lama-generation-as-allocation-2026]
confidence: 0.7
provenance_state: extracted
---

# Token-Level Advertising：LAMA 把拍卖写进自回归生成

## 一句话结论

LAMA（Latent Advertiser Mixture Auction，潜在广告主混合拍卖）把广告主的竞争直接嵌入大模型的自回归解码过程，让内容生成、广告分配与支付结算在同一套序贯机制中共同完成；作者把这个范式概括为 **生成即分配（Generation as Allocation）**——传统拍卖分配的是货架上已经存在的「位置」，LAMA 试图回答的是货架本身如何在生成中长出来。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 问题背景：AI 摘要截断点击，却让广告位变成「生成结果」

Pew Research 跟踪 900 名美国成年人、分析其中 68,879 次 Google 搜索：AI 摘要出现在 18% 的搜索中，一旦出现，用户点击传统搜索结果的比例从 15% 降至 8%，点击摘要所列来源的比例更低至 1%；但点击并非消失，Adobe Analytics 观察到 2024-07 至 2025-05 美国零售网站来自生成式 AI 的访问增长到原来的 35 倍，且这批访问的跳出率比其他来源低 27%、单次停留时间长 38%。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

平台动作已经先行：OpenAI 于 2026-02 在美国面向 Free 与 Go 用户启动 ChatGPT Ads 测试，Google 把 Search/Shopping Ads 扩展进 AI Overviews 并开始在 AI Mode 中测试广告；WPP Media 在 2026 年中期预测中首次把 Generative Search 单列为广告渠道，预计其全球广告收入将从 2026 年的 51 亿美元增至 2030 年的 1000 亿美元以上。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

结构性变化在于默认前提被推翻：过去二十多年的广告机制都建立在「预先定义一个槽位，拍卖只决定归属」之上，而生成式界面是逐字生成内容，某个品牌是否适合出现、出现在哪句话里、以什么方式出现，都取决于回答此前写了什么——广告机会不再只是等待分配的固定位置，它可能是由生成过程本身创造出来的。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## LAMA 机制：四步流程

- **报告**：针对当前文本前缀，每位广告主提交一组「继续往下写的延续价值」（报告的是不同候选 Token 被选中后对未来整段回答的价值，而非给每个 Token 直接报价）；平台将其与原始语言模型的下一词分布结合，得到每位广告主偏好的下一 Token 策略。
- **生成**：平台维护一个关于广告分配的后验概率，每一步先按当前后验选出一位「引导本步生成」的潜在广告主，再从该广告主对应的策略中采样下一枚 Token；边际上使用的分布就是所有广告主策略按当前后验加权得到的混合分布。
- **更新**：新 Token 产生后，平台检查它在各广告主策略下出现的概率，用贝叶斯规则更新后验——某个 Token 越符合一位广告主偏好的生成方向，该广告主被分配的概率就上升。
- **结算**：回答结束后最终后验成为广告分配概率，平台据此抽取一名赢家，赢家获得超链接、可点击回答块或赞助卡片等最终曝光，并按整条生成轨迹结算。

真正要抓住的是「生成状态与分配状态始终绑定」：每写出一个 Token，前缀变化会同时改变各广告主的延续价值与分配后验，所以最终广告机会不是被填满的空位，而是这条生成路径走到终点后形成的结果。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 激励兼容：为什么广告主不能中途「变卦」

机制走到这一步，麻烦才真正出现：只要广告主能够细致地影响生成，就会多出新的操纵空间——例如在开头故意压低价值把回答引向有利方向，等前缀对自己有利后再突然抬价争夺最终曝光；传统一次性拍卖只需管住开场报价，Token 级机制必须保证在回答生成到任何一个可达前缀后，中途变卦都仍然无利可图。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

LAMA 用两层约束封堵这个口子：第一层是「不能随便改口」的**贝尔曼一致性账本**，平台为每位广告主保存当前状态的一个标量价值，广告主下一步提交的整组子状态价值必须能通过软贝尔曼方程聚合回账本中的父状态价值，即新前缀上的局部报告必须与此前承诺相容；第二层是**沿生成路径变化的支付规则**，若某 Token 让一位广告主「后验概率 × 延续价值」的前景改善则收费，若让前景变差则可能补贴，单步转移可以为负，但整条路径的支付像望远镜求和一样前后抵消并在终点完成结算。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

三条结论构成机制的理论保证：**马尔可夫占优策略激励兼容（Markov DSIC）**，即其他广告主采取任意可行策略时，一直诚实的广告主在每个可达前缀继续诚实报告都优于任何后续偏离（比「开场报真话」更强，直接覆盖序贯生成中的中途操纵）；**个体理性与预算平衡**，诚实参与带来非负期望效用，单赢家版本下赢家实际效用非负、未获胜者支付为零，平台在期望意义下弱预算平衡；**接近最优的 KL 正则化福利**，设候选广告主为 n、平衡系数为 β，LAMA 与理想最优解的加性差距不超过 β log n——β 越大回答越贴近自然生成，β 越小广告价值能施加更强影响。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 系统实现：不算整棵树，只记两类量

理论机制落到系统会立刻撞上算力现实：每个文本前缀后面都有词表规模的分支，所有可能回答共同组成一棵巨大的 Token 树，逐分支展开并计算每位广告主的最优延续价值基本不可能。论文的办法是只记住两类量——**根价值**（用户问题刚出现时这次机会对该广告主的整体价值）与**局部优势**（当前前缀下选择某 Token 相对平均延续的价值增减），于是只要知道根价值、再把真实生成路径上的局部优势逐步累加，就能恢复每个已访问前缀的价值，平台无需展开整棵树。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

系统使用一个**共享的广告主条件模型**：广告主身份、创意、落地页与定向信息写入条件提示，模型一端通过偏好数据学习局部优势，另一端接入标量回归头预测根价值，所有广告主共享同一套主干参数而不必各自训练完整模型；广告主只需提供业务信息与最终价值信号，由平台代为生成逐 Token 报告，整套流程更接近现有的自动出价（Auto-bidding）服务。在线服务时系统先用常规广告漏斗召回很小的候选集，再对候选广告主做条件前向计算。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 实验：收入提升 10.7%，用户质量持平

概念验证使用 Webis Generated Native Ads 2024 数据集中 1239 条真实商业搜索查询，覆盖健身（291 条，Bowflex/C4 Sport/ClassPass）、度假（456 条，Expedia/Priceline/Tripadvisor）、汽车（492 条，Kelley Blue Book/Edmunds/CARFAX）三个垂类，参考语言模型为 Qwen3-14B，共享报告模型用 LoRA 训练；需要强调的是实验中的广告价值来自模型预测的展示点击价值而非真实线上点击，用户侧质量按 GEM-Bench 的相关性、信息量、流畅连贯、广告自然度、广告事实性与侵入性等维度评估。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

对比方法覆盖生成前分配、生成后分配、原始回答、编辑插入、广告主策略生成，以及响应级聚合机制 MOSAIC。从均值看 LAMA 四项指标最高：平台福利 0.5205（最佳基线 0.5080）、平台收入 0.8305（最佳基线 0.7501，提升约 10.7%）、广告主价值 0.8568（最佳基线 0.8253）、用户质量 66.5239（最佳基线 66.4785，两者在误差范围内基本持平）——最值得注意的不是「所有数字都更高」，而是收入与广告主价值的提升没有以明显牺牲回答质量为代价。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

出价偏移实验固定其他广告主、只把目标广告主的报告整体上下平移：报得越高，其贡献的 Token 比例、最终分配概率与支付都越高，但按真实价值计算的期望效用恰好在诚实报告处达到峰值——过度抬价能抢到更多影响力，却被更高支付抵消。效率代价方面，三广告主实验显示端到端平均延迟约为自然回答的 3.53–3.94 倍，每 Token 耗时约 3.51–3.57 倍，与「候选广告主数 + 1」的理论计算量基本一致。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 边界与未解问题

论文只研究单次、单赢家机会，尚未覆盖多赢家、跨请求预算、ROI 约束与长期竞价；精确的激励保证依赖精确报告，有限数据训练出的模型只能近似恢复这些报告；广告价值来自离线模型预测而非真实投放中的点击与转化；三广告主设置已带来约 3.5–3.9 倍端到端延迟，候选集扩大后的系统优化仍是关键。同时，真实性、披露、隐私、品牌安全与用户信任并不会由拍卖机制自动解决，机制只负责后台的内容生成、分配与支付，前台如何披露广告、保护用户信任仍要靠产品政策与安全系统共同完成——因此这项工作的价值更像一张新地图，而不是一套可直接上线的广告产品。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 对 Agent 系统的迁移含义

论文自己点出了智能体方向的延伸：当智能体执行一条多步流程时，每一步都可能调用不同的付费工具，未来的「赞助工具选择」也许不再是静态列表，而是嵌入决策轨迹的序贯机制。把这条线索与 LAMA 的机制结构并置，可迁移的是一类通用问题——**当生成轨迹本身被多个有利益的主体竞相影响时，如何设计一套在轨迹任意可达状态下都保持诚实最优、且能把支付沿轨迹望远镜式抵消的规则**；这与当前 Agent 生态里工具市场、多 Agent 协作中的资源分配与结算问题同构，也是 [[concepts/tool-use-reasoning|工具调用推理]] 在「多主体竞争」维度上的一个机制设计切面。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

该工作与既有的 Agent 经济与支付协议研究构成上下游：[[entities/agent-capital-markets-wright-shensiquan|Agent 资本市场]] 讨论自主 Agent 如何被融资，[[entities/decisions-and-dollars-agent-economics-data-fintech|Agent 经济学]] 讨论决策与结算的经济学基础，[[entities/ap2-agent-payments-protocol-hands-on-analysis|AP2 协议]] 与 [[entities/agentcore-payments-x402-agentic-commerce|x402 Agentic Commerce]] 提供支付轨道，而 LAMA 处在前一环节——**在生成发生的那一刻决定「谁被听见、谁买单」**；相较之下 [[entities/ai-commerce-brand-vs-marketplace-founder-park|AI 电商：品牌 vs 平台]] 讨论的是商业形态之争，LAMA 讨论的是形态未定之前机会如何被创造。^[raw/articles/token-level-advertising-lama-generation-as-allocation-2026.md]

## 相关实体

- [[entities/agent-capital-markets-wright-shensiquan|Agent 资本市场：自主 Agent 融资框架与批判]] — 自主 Agent 的经济主体化
- [[entities/decisions-and-dollars-agent-economics-data-fintech|Agent 经济学：决策与美元]] — 决策轨迹上的经济结算
- [[entities/ap2-agent-payments-protocol-hands-on-analysis|AP2 Agent 支付协议实操分析]] — Agent 支付轨道
- [[entities/agentcore-payments-x402-agentic-commerce|AgentCore × x402 Agentic Commerce]] — Agent 商业结算基础设施
- [[entities/ai-commerce-brand-vs-marketplace-founder-park|AI 电商：品牌 vs 平台]] — 商业形态之争
- [[concepts/tool-use-reasoning|工具调用推理]] — 多步决策轨迹中的工具选择
- [[concepts/speculative-decoding|投机解码]] — 解码阶段的效率—质量权衡（本机制的延迟代价与之同类）

→ [[raw/articles/token-level-advertising-lama-generation-as-allocation-2026|原文存档]]
