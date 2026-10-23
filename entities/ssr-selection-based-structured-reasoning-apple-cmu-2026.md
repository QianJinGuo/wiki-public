---
title: "SSR（Selection-based Structured Reasoning）：把 Agent 推理从逐 token 生成改成「做选择题」"
created: 2026-10-19
updated: 2026-10-24
type: entity
tags: [agent, reasoning, multimodal, inference-optimization, apple, tool-use]
sources: [raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026]
confidence: 0.7
provenance_state: extracted
---

# SSR（Selection-based Structured Reasoning）：把 Agent 推理从逐 token 生成改成「做选择题」

苹果与卡内基梅隆大学提出的 SSR 框架，把 ReAct 式 Agent 每步的 free-form thinking 替换为「从预定义推理库中选一条」，在 7 个多模态搜索 benchmark 上把每轮推理延迟压掉 90% 以上，同时保持与自由生成相当的成功率。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## 动机：小模型的「废话文学」

多数搜索 Agent 遵循 ReAct 的观察-推理-动作循环，每步动作前先生成一段自由形式推理；2B/4B 端侧小模型偶尔生成很长的推理，但对下一步动作与最终决策没有实质帮助。论文分析大量搜索轨迹后发现：虽然题目实体、查询、观察千差万别，决定下一步动作的高层推理逻辑却高度重复——「某场 AI 会议在哪里举办」和「LEGO 是哪年成立的」共享同一条逻辑：实体已知但目标属性缺失，就用实体名加属性去搜索。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

这直接推出 SSR 的核心假设：搜索 Agent 的推理空间不必覆盖整个自然语言空间，可以压缩为一组可复用的策略选项。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## 方法：推理库 + 选择式推理

推理库共 6 条选项：不确定图中实体时用以图搜图识别；需要具体事实时用文本搜索查询；需要细节时裁剪或放大图像局部；证据已收集时基于证据作答；图中就有答案时直接读图作答；常识可答时直接回答。每轮交互中，Agent 不再逐字生成推理，而是先选一条推理作为 rationale，再基于它生成实例相关的动作细节（搜索 query、裁剪框等），从而把「为什么这么做」与「具体怎么做」解耦。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

选择机制不需要额外分类头：直接让 LLM 自己当分类器——对每个推理选项计算其在当前 context 下的 token 平均似然，经 softmax 得到推理库上的类别分布后采样。打分与动作生成共享同一套模型参数与 system prompt，也不需要 SFT warm start。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## Parallel Reasoning Decoding：效率优势的来源

由于所有推理选项都是预定义的，打分可以完全并行：token 层面通过 teacher forcing 一次性 prefill 整条选项，在一个 batched forward pass 中算完所有 token 似然；选项层面所有选项共享同一份上下文 KV cache 同时打分。整个推理过程模型不需要自回归生成任何新 token。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

论文给出的直观对照：4B 模型对一张爱丁堡大学建筑照片生成了 156 个 token 的冗长思考、耗时 1821.92ms，还把建筑误判为苏格兰国家博物馆并导向错误搜索方向；SSR 仅用 32.66ms 就选中「通过图像识别实体」并直接触发图片搜索。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## 训练接口与实验结论

SSR 是通用推理接口替代方案，可接入现有 SFT、RL 管线；由于推理被约束在推理库内，可以直接上 RL 而不必先用 SFT 教会模型「特定的思考方式」，省掉 SFT 数据构造与训练时间，行为也更可预测。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

在 Qwen3-VL 2B/4B、H100 + SGLang 上评估 MMSearch、HR-MMSearch、FVQA-test、InfoSeek、SimpleVQA、LiveVQA、MAT-Search 共 7 个多模态搜索 benchmark：4B SSR（GRPO）平均成功率 61.37%，与同规模 SOTA 持平；2B SSR 达 51.26%，堪比零样本 8B 模型；训练方法换成 GSPO / SAPO / SFT 时成功率也与 free-form 版本相当。效率上每轮推理延迟降低超 90%，每题模型总推理延迟降低 28–54%，p95 推理延迟仅 0.07 秒（baseline 为 0.8–1.9 秒）且更稳定。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## 消融实验

只让模型选择编号（如「[1]」）而非完整文本，平均成功率下降 3.9%，且 7 个 benchmark 全面受损——说明打分确实在调用模型的语义理解能力做上下文匹配，而非学一个纯索引分类器。推理库规模有单调收益：从 1 条通用选项增加到 6 条，平均成功率从 37.24% 单调提升到 61.37%，其中 1→2 条（区分「作答」与「调用工具」）就带来 17.27 个百分点跃升。选择也是自适应的：同一 checkpoint 在偏简单事实问答的 SimpleVQA 上「基于图片作答」占比最高，在高分辨率图片为主的 HR-MMSearch 上「检查图片细节」占比最高。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## 边界与意义

SSR 更应被理解为一次推理空间的重新组织：从完全开放的语言空间，转为紧凑、可复用、语义明确的策略空间。开放式数学推理、定理证明、代码设计仍需要 free-form 表达能力；但在多轮搜索与工具使用场景中，每步往往只有少数几类高层决策，策略层推理不必每次在完整语言空间中重新生成。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

对 Agent 工程的启示：把「每步推理」从算力瓶颈改成查表式选择，是端侧小模型跑 Agent 的一条现实路径，其代价是推理表达力被限制在预定义策略集合内——库的设计质量直接决定上限。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md]

## 深度分析

### 自由形式 CoT 为什么在小模型上是冗余的

ReAct 每步的 free-form 思考，实际承载的决策信息带宽很窄：判断「信息够不够、该调哪个工具」。2B/4B 模型把有限表达能力花在复述观察上（「这张图像某个博物馆」），产出的是低熵重述，不是新增信息。更关键的是收益与风险不对称——156 个 token 的思考不但没帮上忙，还凭空造出一个错误实体（苏格兰国家博物馆）并锁死了后续搜索方向：自回归生成本身就是一个新的幻觉面，错误还会在 token 链上复合。SSR 的选择只做一次前向，误差不沿序列累积，等于把测试时算力从「序列深度」换成了「上下文匹配宽度」。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md] 但前提是：搜索 Agent 的高层决策空间本来离散且小；这不是对 CoT 的普遍否证。

### 覆盖度与表达力的取舍

消融里库规模 1→6 条带来 37.24%→61.37% 的单调提升，说明覆盖度直接决定上限；但边际递减明显——1→2 条（切「作答/调工具」）一次拿走 17.27 个百分点，其余四条合计只补了约 6.9 点。这与决策树的信息增益同形：第一刀砍掉最大不确定性，后面只是细分。表达力的让渡被切成两半：策略层被锁死（库外策略无法表达，模型只能退化到最近邻选项，形成「策略错配」），执行层（query、crop 坐标、最终答案）保留自由生成——正因为只搬走策略层，成功率才没掉。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md] 另外，库不是免费的：选项越多，语义区分度要求越高，重叠会让 softmax 变平、选择抖动，且每条选项都要预填充一遍，总算力是 O(k·len)。

### 并行选择为何拆掉串行延迟墙

自回归 decode 是访存受限：每个 step 都要把权重搬一遍，batch=1 时 GPU 利用率极低，156 个 token 就是 156 次串行搬运，1821.92ms 里绝大部分是带宽开销而非计算。SSR 的打分是预填充形态，计算受限：整条选项一次前向，所有 token 并行，各选项共享同一份 context KV cache、只在 batch 维拼接，于是延迟约等于一次预填充的固定成本（32.66ms），与「思考长度」基本解耦。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md] 这改变的是延迟分布的形状而不只是均值：p95 从 0.8–1.9 秒压到 0.07 秒，长尾消失。对多轮 Agent 循环，尾部延迟才是体验的决定项，因为每轮 p95 会累加。代价是成本从串行时间转为并行算力——库变大时预填充开销会回来，与 [[concepts/speculative-decoding|投机解码]] 的分野也在此：后者在生成路径上加速，SSR 取消生成。

### 消融的隐含结论与失败模式

「只选编号掉 3.9%、7 个基准全面受损」是最有信息量的结果：增益来自模型读选项文本并与上下文做语义对齐，而不是学了个位置分类器——选项措辞因此是超参，改文案等于改行为。另一条更硬：自回归版必须靠 SFT 才守得住格式，而仅用格式奖励的 RL 约束不住「乱想」——软约束与机制级硬约束的可靠性差一个量级，SSR 本质上是把一个对齐问题转成了架构问题。^[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026.md] 附带的好处是每步策略标签天然可审计。已知失败模式有三类：需要组合两个策略的单步任务（先裁剪再以图搜图）无法表达；图文混合输入下「文本搜索」与「以图搜图」边界模糊；罕见选项似然系统性偏低，形成自我强化的选择偏科——论文展示的自适应分布，可能一部分只是训练分布的镜像，而非任务最优。库从哪来（人工还是轨迹挖掘）、跨域怎么重建（GUI 智能体的决策类几乎完全不同：点击、滚动、输入，库不可直接复用）、库固定后更强模型是否被压制，文中都未回答。

## 实践启示

1. **先量决策熵，再决定要不要选择式。** 抽 50–100 条真实轨迹标注每步高层决策类别；若类别 ≤10 且前 3 类占七成以上，收益大概率成立；若每步都不同，选择式会立刻撞上库外失败。
2. **第一刀切「作答/调工具」，别一上来就切细。** 消融显示 1→2 条即得 17.27 个百分点，先把动作空间最大的二分做对，再按错误分析逐条加；每加一条都要跑全量回归，边际收益衰减很快。
3. **选项写成「条件 + 动作」的自然语言句，不要写成标签。** 语义打分依赖文本自带的匹配信号，编号会掉 3.9%；把选项文案纳入 prompt 管理：版本化、A/B、不随手改。
4. **盯 softmax 分布，而不只是选中项。** 记录每轮熵与 top1–top2 间隔：熵长期偏高说明选项语义重叠、需重写；top1 长期钉在一项说明它在吞掉流量。这几乎是零成本的可观测性。
5. **用「库外对照」防探索饥饿。** 抽 5% 流量放行 free-form 并对比成功率；若它明显更好，说明库缺策略，该加选项而非调参。RL 阶段给低频但合理的选项加覆盖奖励，避免似然先验把分布锁死。
6. **只替换策略层，保留执行层的自由生成。** query、坐标、最终答案继续由模型生成；若连 query 也改成选模板，会一次性丢掉实例适配能力——这是最常见的过度收敛错误。

## 关联

- 概念：[[concepts/inference-optimization|推理优化]]、[[concepts/tool-use-reasoning|工具使用与推理]]
- 相关实体：[[entities/colt-eccv-2026-latent-thought-chain-multimodal-reasoning|潜空间思维链多模态推理]]、[[entities/sglang-inference-deployment-practice-benchmark-tuning|SGLang 部署实践]]、[[entities/minimax-h3-h200-sglang-benchmark-2026|MiniMax H3 SGLang 基准]]

→ [[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026|原文存档]]
