---
title: "SSR（Selection-based Structured Reasoning）：把 Agent 推理从逐 token 生成改成「做选择题」"
created: 2026-10-19
updated: 2026-10-19
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

## 关联

- 概念：[[concepts/inference-optimization|推理优化]]、[[concepts/tool-use-reasoning|工具使用与推理]]
- 相关实体：[[entities/colt-eccv-2026-latent-thought-chain-multimodal-reasoning|潜空间思维链多模态推理]]、[[entities/sglang-inference-deployment-practice-benchmark-tuning|SGLang 部署实践]]、[[entities/minimax-h3-h200-sglang-benchmark-2026|MiniMax H3 SGLang 基准]]

→ [[raw/articles/ssr-selection-based-structured-reasoning-apple-cmu-2026|原文存档]]
