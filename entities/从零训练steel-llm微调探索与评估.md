---

title: "【从零训练Steel-LLM】微调探索与评估"
type: entity
created: 2026-07-04
updated: 2026-09-21
tags: [wechat, ai]
rating: v8c7
sources:
  - raw/articles/从零训练steel-llm微调探索与评估
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 【从零训练Steel-LLM】微调探索与评估

**来源**: 炼钢AI（作者知乎「战士金」） · **发布日期**: 2024-10-24

**原文链接**: https://mp.weixin.qq.com/s/KK0G0spNw0D9rPUESkHMew

---

## 摘要

「从零训练 Steel-LLM」系列中讲 SFT 与评测的一篇：基座是约 1B、预训练 1060k step、中文语料约占 80% 的自训模型，目标是同时提升对话与选择题作答能力。作者用 BAAI Infinity-Instruct 中文子集（约 70w 条）、wanjuan 中文选择题、Better-Ruozhiba 与自我认知数据做 SFT，最终 CEVAL 38 分、CMMLU 33 分。真正的价值在于四组对照实验——全量数据 vs 中文筛选、是否把 CMMLU 测试集混入训练、COT vs 非 COT 格式，以及准确率随微调 step 的边际收益曲线。^[raw/articles/从零训练steel-llm微调探索与评估.md]

## 核心要点

- 项目起点是朋友闲置的单卡 A100 80G SXM，一个多月后机器被收回，只训到 20k step（原计划 100k），后靠某 top 3 老师资助的 H 系列机器补足算力，2024 年 8 月前后完成，全程业余时间推进。
- Infinity-Instruct 7M 全量（约 700w 条）混入训练时 CEVAL 只有 30 出头；去掉约 630w 英文数据、只留约 70w 中文后升到 38 分，这是最好的一版微调模型。
- wanjuan 中文选择题早在预训练数据里混入过（`prepare_steel_llm_data.py` 有专门逻辑），但模型续写与对话都不稳定、选择题也答不好，属于「四不像」，于是微调阶段回炉并规范格式；其 `answer_detail`（解析）字段被作者视为关键，没有解析就近乎让模型背答案。
- 微调统一口径：加载 1060k step 预训练 checkpoint，3 个 epoch，学习率 2e-5，micro batch 8 × 梯度累积 8（等效单卡 batch 64）；每组实验固定混入少量 Better-Ruozhiba 与自我认知数据。
- 实验1（全量 Infinity-Instruct + 全量 wanjuan exam）：CEVAL 随 step 上涨，约 18000 step 后走平，最好约 32%；作者据此判断小模型即使预训练见过 wanjuan exam 仍会遗忘，多喂微调数据依然有益。
- CMMLU 刷榜实验：训练集加入 CMMLU 时 CMMLU 得 36%，不加入约 33%，但 CEVAL 没有提升。
- 实验4（COT）：让 wanjuan exam 按「先解析再答案」微调，CEVAL 反比非 COT 更差，作者归因于小模型推理能力不足。

## 深度分析

### 微调数据构造：语种分布对齐先于数据量之争

数据由四块拼成。BAAI/Infinity-Instruct 的选取标准是「大机构发布、版本要新」（机构要面子、质量相对可控），且该数据集自带语言标签与任务类型元信息，可用于切分；wanjuan 选择题带 `q_type / std_ans / answer_detail / major` 等字段，是可直接改造成 SFT 样本的结构化数据；Better-Ruozhiba 在原始「弱智吧」数据上逐条人工审阅，剔除格式错误、改写部分答案；自我认知数据取自 EmoLLM 的 `self_cognition_EmoLLM.json`，只把「心理健康助手」换成「Steel-LLM」。^[raw/articles/从零训练steel-llm微调探索与评估.md]

分水岭在语种筛选：700w 条「一把梭」只换来 CEVAL 30 出头，统计后发现中文仅约 70w、英文对话约 630w，而基座预训练 80% 是中文。切回 70w 中文后直接到 38 分，说明**微调数据的语种/领域分布必须与基座预训练分布一致**。作者还顺带回应「微调能否注入知识」：微调与预训练一样是 next token 训练，当然能注入知识；但「少量数据换来好 SFT 模型」的前提是基座足够强，在弱基座上他选择百万级 SFT 数据量。^[raw/articles/从零训练steel-llm微调探索与评估.md]

### SFT 超参、step 边际收益与灾难性遗忘的实证

微调口径很朴素：预训练 checkpoint 起步，3 epoch，lr 2e-5，micro batch 8 配 8 步梯度累积（等效 batch 64）。实验1 给出的边际收益曲线很清晰——CEVAL 随 step 上升，但 18000 step 后基本走平，最好约 32%。在 1B 规模上，SFT 的收益窗口明显早于 epoch 边界，把早停绑在「训满 3 个 epoch」并不合理。^[raw/articles/从零训练steel-llm微调探索与评估.md]

遗忘结论同样值得记录：wanjuan exam 在预训练阶段已见过，按常理「学过就不会忘」，但模型小、容量有限，训练过的内容仍会丢掉一部分，所以微调继续喂仍有收益，而重放（replay）是最省事的缓解手段。实验3 的意外副产品从反面印证收益窗口：因作者疏忽，该实验加载的是实验1 训出的 checkpoint 而非原始预训练模型，结果 CEVAL 没有进一步增长，等于一次天然的继续训练对照。^[raw/articles/从零训练steel-llm微调探索与评估.md]

### 榜单过拟合能涨多少分：CMMLU 刷榜实验

作者受天工大模型技术报告与论文《Training on the Benchmark Is Not All You Need》启发——两者都指出部分榜单前列模型存在直接过拟合测试集的行为——于是把 CMMLU 数据加进训练集做消融：加入后 CMMLU 测试 36%，不加入约 33%。结论两层：过拟合测试集对刷榜确实有效，但对小模型作用有限，「死记硬背都不能特别好记下来」；且对 CMMLU 的过拟合不能提升 CEVAL 分数，两套榜单题型大部分相同（STEM、social science 等）却无法互相迁移，说明小模型上过拟合测试集的泛化性一般。^[raw/articles/从零训练steel-llm微调探索与评估.md]

方法缺口也交代了：对 CMMLU 做 SFT 时只让模型预测选项（拿不到解析），作者猜测若学「选项 + 解释」会涨更多——只喂答案标签才涨 3 分，而现实中评测污染常连同推理链一起泄漏。^[raw/articles/从零训练steel-llm微调探索与评估.md]

### COT 格式失败：格式是数据分布的一部分

实验4 在实验1 数据配置上把 wanjuan exam 改写成 COT 形式（先「解析」后答案），非 COT 则先答案后解析，结果 COT 的 CEVAL 反比非 COT 差一些，作者解释为小模型推理能力不足、多出的解析生成带来噪声。构造细节值得一提：COT 版本用正则（`故选.`）切掉 `answer_detail` 中泄露答案的句子、末尾统一追加「答案为 X」，本质是把「解释」与「答案」解耦，防止模型从解析里抄答案。^[raw/articles/从零训练steel-llm微调探索与评估.md]

作者还对某大厂 1.5B 模型做了交叉验证：要求「先输出解释再输出答案」，但很多情况下模型仍先给答案后给解释，或干脆只给答案，因此推断其 SFT 数据格式本就是先答案、并未使用 COT。这是可迁移的探测手段——**把模型的输出格式当作窥探其 SFT 数据分布的窗口**。对 1B 级模型结论务实：先对齐答案格式比追求推理过程更划算。^[raw/articles/从零训练steel-llm微调探索与评估.md]

### 单卡预算下的工程取舍，与预训练阶段的口径衔接

所有决策都能追溯到算力预算：单卡 A100 起手、一个多月后被收回、靠资助机器续命、业余时间推进——这解释了微调阶段为何不做大规模超参搜索，而是以固定口径跑少量对照实验，把变量集中在数据组成与格式上。这也是小团队做 SFT 的理性策略：数据组成的不确定性远大于学习率的小数点。^[raw/articles/从零训练steel-llm微调探索与评估.md]

与预训练的衔接还体现在损失函数之外。预训练数据里混入 BELLE、moss 等对话数据，本意是让 base 模型自带对话能力，实际却是续写与对话都不稳定、选择题也答不好——「在预训练里掺 SFT 数据」不能替代真正的 SFT 阶段，反而污染基座续写能力；而 [[entities/从零训练steel-llm模型设计|模型设计篇]] 上传的中间 checkpoint 正是本篇全部实验的起点。case 展示给出能力边界的第一手证据：模式化任务（自我介绍、快速排序、写诗）尚可，翻译达意但磕巴（英文数据仅 20% 的后果），而三位数加法常误差 10 以内、数兄弟姐妹漏算自己——数值计算与多步严格推理最薄弱。作者坦承 RL 对齐尚未做，后续聚焦 SFT 样本筛选并同步代码到 GitHub（`zhanshijinwat/Steel-LLM`），使上述结论可复现。^[raw/articles/从零训练steel-llm微调探索与评估.md]

## 实践启示

1. 微调前先统计微调数据与预训练数据的语种/领域分布，把分布对齐当第一优先级——700w→70w 中文切片换来 CEVAL 6 分，成本几乎为零。
2. 优先选大机构发布、版本较新、带语言标签与任务类型元信息的指令数据集，元信息是做切片筛选的前提。
3. 结构化选择题数据要保留「解析」字段，并在构造时把「解析」与「答案」解耦（如正则切掉泄露答案的句子），避免模型学会抄答案而非解题。
4. 不要把 SFT 收益默认绑在 epoch 上：画出准确率-微调 step 曲线找出平台期（本例约 18000 step）并据此早停；小模型遗忘明显时，用重放缓解比换复杂方法更省算力。
5. 自评分数要与外部榜单交叉验证：若训练集混入测试集数据会涨分（本例 +3 分），此时必须检查同类题型的其他榜单是否同步提升，不同步即为过拟合信号。
6. 不要在基座推理能力不足的小模型上照搬 CoT 训练范式：先保证输出格式正确，再谈推理过程。

## 关联

- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- [[entities/从零训练steel-llm模型设计|【从零训练Steel-LLM】模型设计]] — 前序篇章：本篇微调实验的起点即该篇训练的 1060k step checkpoint
- [[concepts/llm-pretraining-vs-sft|预训练 vs SFT]] — 「预训练注入知识、微调学习对话方式」的分工在此被实测检验
- [[concepts/catastrophic-forgetting|灾难性遗忘]] — 1B 小模型在 SFT 中遗忘 wanjuan 预训练数据的直接证据
- [[entities/sft-data-preparation-advanced-strategies-2026|SFT 数据准备进阶策略]] — 微调数据筛选、去重与质量阈的延伸方法

→ [[raw/articles/从零训练steel-llm微调探索与评估|原文存档]]
