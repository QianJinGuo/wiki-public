---
title: "Magic 10x More Efficient Pretraining (Magic Lab)"
created: 2026-09-10
updated: 2026-11-18
type: entity
tags: [pretraining, scaling-laws, training-efficiency, llm, magic, frontier-lab]
sources: [raw/articles/magic-10x-more-efficient-pretraining]
confidence: 0.7
---

# Magic: 10x More Efficient Pretraining

Magic（2026-09-08 研究更新）宣称其预训练配方已实现比主流开源 base model 高 >10x 的算力效率：以约 DeepSeek V4 Pro Base 1/50 的 FLOPs 匹配其能力（约等于 GPT-3 预训练算力的一半，GB200 上约 $0.5M），继续 scale 10x（~$4M）后在 perplexity evals 上明显超越所有公开开源 base model。按图 1 的 scaling laws，同能力在 DeepSeek V4 Pro 配方下成本 >$100M。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 方法论可信度信号

文章给出多个支撑其效率声明的工程细节：167 个域上的 scaling-law 拟合、每次改动 3 个模型的多尺度评估、heldout 去污（decontamination）协议、以及跨引擎 logprob 验证（防止评估引擎差异污染对比）。模型与评测均为私有，无法独立复现——这是实验室研究公告与可复现论文之间的固有差距。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 战略定位

Magic 的路线声明是"预训练 + agentic RL + long-context 足以构建超人 coding agent 并自动化 AI 研究"，本文聚焦预训练一环。其效率优先路径（算法效率替代 10 万卡集群）对算力受限实验室有参考意义，与 [[concepts/scaling-laws|Scaling Laws]] 的"算力即一切"叙事形成对照：效率改进可以数量级地移动成本曲线。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 深度分析

">10x" 不是单点测量，而是跨评测的保守下限。图 1 中各 eval 的 effective-compute multiplier 区间极宽——在 heldout 数学上对 DSv4 Pro 高达 127x，对 Nemotron 3 Ultra 仅 15x；图 3 的 167 域分解进一步显示增益高度域条件化：在 SWE/AI R&D 等目标域可达数十倍，而在 local news、law 等被刻意降权的域上乘数甚至低于 1x。因此效率声明的准确读法是：Magic 把数据配比与目标能力（coding、AI 研究）对齐，在偏好域上集中兑现算力红利，并明确接受在边缘域被开源 base model 反超的代价。^[raw/articles/magic-10x-more-efficient-pretraining.md]

效率来源是复利式工程而非单一大注。相对自身 V2 的效率轨迹（1.9x → 11x → 24x，累计 505x）表明进步来自架构、optimizer、训练目标与数据策展数十项改动的乘积；支撑这一复利的是评估纪律——每项候选改动都训练 3 个跨越 2 个数量级 compute 的模型，只有幂律拟合外推显示在 hero scale 仍然有利才保留，且每几周做 1/10 hero scale 中试、每几个月做全量 hero run。同样值得注意的是反向结论：NanoGPT speedrun 式小模型技巧大量不迁移到大模型，部分主流 LLM 特性甚至可以删除而不损害大规模性能。对算力受限实验室而言，真正的护城河是"研究基础设施 + 规模化验证流程"，而非某个 secret sauce。^[raw/articles/magic-10x-more-efficient-pretraining.md]

私有声明的不对称可验证性是其可信度设计的关键。模型与数据私有，无法复现，但协议细节封堵了主要的不信任路径：heldout 去污只对自己模型执行，而开源模型一侧的潜在污染只会高估对手成绩（有利于基线的偏差方向）；logprob 对比同时在 vLLM、SGLang、GB200/GB300 与 Fireworks 第三方引擎上交叉验证，过程中确实发现了推理后端 bug；评测集用与训练管线不同的 parser/OCR 构建，防止模型学到训练解析器特征后虚低 loss。这是一种"发布协议而非权重"的公告模式——结论不可复现，但产生误导的每条已知路径都被逐一处理。^[raw/articles/magic-10x-more-efficient-pretraining.md]

预训练增益向后训练的传导被显式验证过。针对"bpb loss 改进未必兑换成下游能力"的常见质疑，Magic 从 base model 直接起跑（无 SFT、无 distillation）做了一次 16k CoT 预算的短 math RL：V5 e24 在约 0.2% 预训练算力的 RL 预算下 AIME26 pass@1 越过 90%，heldout 数学集 72% 对比 DSv4 Pro 的 65%，且 pass rate 仍在对数线性攀升。这把 [[concepts/llm-pretraining-vs-sft]] 的分工叙事具体化——预训练阶段的效率乘数在后训练阶段被继承而非稀释。同时 Footnote 提示 AIME 在部分第三方模型中已发生污染，基准去污是这类跨实验室对比的普遍陷阱而非 Magic 独有的顾虑。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 实践启示

1. 算力受限团队的效率路线是"数十项小改动的乘积 + 强制规模化验证"，不是寻找单一 silver bullet；把"每次改动训练 3 个跨越 2 个数量级的模型、幂律外推决定去留"作为制度化流程，而不是临时做法。
2. 评估卫生应作为基础设施投资：tokenizer 无关的 bits-per-byte 指标、heldout 去污、跨引擎 logprob 交叉验证、评测 parser 与训练 parser 分离。这些协议既是可信度信号，也真实拦截了推理后端 bug 这类会扭曲结论的误差源。
3. 效率改进是域条件性的：先明确目标能力（如 coding 与 AI 研究），再把数据配比向其倾斜，并公开承认在降权域上的劣势——图 3 中低于 1x 的域正是这种有意取舍的记录。
4. 小模型上的结论不可直接外推：NanoGPT speedrun 改进大量不迁移到大规模；这与 [[entities/pretraining-progress-is-mostly-data|数据轴分析]] 一致，效率大头往往在数据策展与配比，而非架构花活。
5. 阅读同类实验室声明时，先找逐评测 multiplier 分解与协议细节，再看头条数字：">10x" 在偏好域上可达上百倍，在降权域上可能低于 1x，单看 headline 会同时高估与低估实际位置。

## 关联

- 预训练数据/规模研究：[[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|Scaling Domain Data Repetition]]
- 预训练 vs 后训练分工：[[concepts/llm-pretraining-vs-sft]]
- 数据 vs 模型轴的效率分解：[[entities/pretraining-progress-is-mostly-data|Pretraining Progress Is Mostly Data]]

→ [[raw/articles/magic-10x-more-efficient-pretraining|原文存档]]
