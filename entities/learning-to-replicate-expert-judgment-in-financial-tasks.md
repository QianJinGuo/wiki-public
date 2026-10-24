---

title: "Learning to Replicate Expert Judgment in Financial Tasks - Thinking Machines Lab"
created: 2026-07-03
updated: 2026-10-24
type: entity
tags: [ai, llm, fine-tuning, finance]
sources: [raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Learning to Replicate Expert Judgment in Financial Tasks - Thinking Machines Lab

## 摘要

Bridgewater AIA Labs 与 Thinking Machines 用资深投资人标注的数据在 Tinker 上微调 Qwen3-235B，把"金融文档信息筛选"训练成专用模型。它在六项投研任务上平均准确率达 84.7%，超过所测最佳前沿模型的 78.2%，推理成本仅为其 1/13.8。作者据此提出"差异化智能"：面向组织特定需求定制的模型可同时在准确率与成本上击败通用前沿模型。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]

## 核心要点

- **六项任务**：金融文章相关性、央行文档利率信号、通用文档与提问匹配度、临时内容标注（区分模板与一次性分析并定位末页）、文档截断点、邮件截断点；前三项看 F1 与准确率，后三项看精确匹配。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]
- **前沿模型近乎抛硬币**：仅给任务描述的朴素 prompt 下，Gemini / Claude / GPT 各版本平均准确率 47%–50%（Opus 4.6 为 47.2%，Gemini 3.1 Pro 为 50.1%）。
- **专家 prompt 的尽头**：由投资人重写指令、并把文章分类拆成"相关且有趣 / 相关但无趣 / 不相关"三档后，准确率升到 74%–78%（Opus 4.8 达 78.2%），仍全部低于业务要求的 80% 信任阈值；自动 prompt 优化再无增益。
- **数据先于算法成为瓶颈**：最初采购供应商的非专家标注，训练后依然糟糕；查推理轨迹发现标签本身就大量错误。
- **专家预算的杠杆用法**：在非专家数据上训练，再用该模型评测自身训练集，凡模型答案与标签冲突的样本送专家复核（模型复现不了自己训练过的样本，意味着样本太难或标签错），清洗后另留测试集做终评。
- **RL 基线**：Qwen3-235B 原准确率 44.8% / 正类 F1 55.24%；接入标准 GRPO 与重要性采样损失后升至 73.48% / 88.95%，仍未过 80%。
- **三项配方**：任务间轮转的交错批处理比完全混合批提升 12.1%；CISPO 损失配非对称裁剪比重要性采样基线提升 10.1%；带强教师的策略蒸馏（OPD）比冻结基座教师再提 3.1%。
- **成绩与消融**：完整配方 84.66% / 92.99%，相比 78.2% 基线错误率降 29.8%，单任务推理成本降 13.8 倍。留一法：去掉交错批处理 72.18%，去掉 CISPO 74.56%，去掉 OPD 72.39%，保留 OPD 但取消"验证新高才升级教师"门控为 81.55%。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]

## 深度分析

### 专家判断为何压不进 prompt

本文最硬的贡献不是"专用模型赢了"，而是标定了 prompt 工程的天花板。朴素 prompt 下 50% 恰好是二分类随机水平，说明模型对"什么对投资人重要"毫无先验。专家 prompt 买到的 28 个百分点是**可被言说的部分**：任务定义、评价维度，以及标签本体的修正。三档标签的重构尤其说明问题——"相关"与"有趣"本是正交维度，强行压成 yes/no 让模型无从下手，很多所谓判断错误其实是标签体系自身的错误。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]

剩下 22 个百分点是默会知识：为什么格陵兰岛的地缘新闻对美股宏观投资人不重要，而对华关税威胁至关重要——两者表面都含"地缘政治"与"金融"。投资人写不出这条规则，因为他们自己也不靠规则判断。GPT 5.4 比 5.2 贵 43% 而准确率几乎不动，说明这道鸿沟不是更多参数能填的：它缺的是**任务特定的 taste 分布**，不是算力。

### 三项配方各买到什么

**交错批处理买的是任务隔离下的表示共享。** 完全混合批让六个任务的梯度在同一批内互相冲淡，逐任务串行又丢迁移、易遗忘；轮转交错（每批一任务、轮流）保住单任务梯度纯度，同时让共享表示持续被复用。它在消融中单项贡献最大（84.66% → 72.18%）。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]

**CISPO 的非对称裁剪买的是错误代价的不对称。** 漏掉重要信息让投资人错过交易，多标一条噪音只浪费三十秒。标准重要性采样的对称信任域假设两类错误等价；非对称裁剪在有利方向放大步长、有害方向收紧，把容量倾斜到高代价错误。

**OPD 买的是把强模型当正则项而非答案。** 其 advantage 为 $r = \text{reward} - \beta \cdot \operatorname{avg}(\text{student\_lp} - \text{teacher\_lp})$ 再去均值化，学生偏离教师分布即受罚。关键在每 20 步把当前 checkpoint 升为教师，但**仅在验证准确率创新高时**才升级，从而避开自蒸馏最典型的退化：教师被自己的错误污染，学生朝更弱分布收敛。消融证明这层门控才是主体——保留 OPD 却去掉门控，成绩从 84.66% 掉到 81.55%，几乎等于把 OPD 的收益全部退回。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]

### 成本曲线与部署经济学

13.8 倍成本差不是"小模型便宜"的常识，而是任务结构决定的。信息筛选**高频、低单价、可批量**：一位投资人每天上百份文档，机构规模乘交易日就是百万级调用，单位成本直接决定能否上线。准确率 84% 但每千任务花几十美元的方案，与 78% 的 API 方案，在预算约束下根本不是同一个产品。

更关键的是准确率与成本在此**同向而非权衡**：235B 开源权重跑在自有推理栈上，既便宜一个数量级，又因领域微调而更准。这打破了"更强=更贵"的假设——若每个业务单元都能用一小批专家标注换一个"更准且更便宜"的模型，采购通用前沿 API 就不再是默认选项。^[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks.md]

### 对领域专家模型的普遍含义

边界要说清：六项任务全是**判别式**的（分类、定位、排序），不涉及生成式判断；公开的仅是被清洗获准发布的数据子集；80% 是 Bridgewater 内部的信任标准，换组织、换任务族会变。

可迁移的方法论相当硬：**领域 alpha 不在公开数据里，而在专家如何过滤公开数据里**。公开信息人人可得，模型间的差异只能来自对"什么值得看"的编码，而编码 taste 的唯一通道是高保真专家标注——不是专家写的规则，是专家做出的具体判断。供应商的非专家标注也败在此处：它们捕捉"文字在讲什么"，任务要的是"文字对这个人此刻是否重要"。

## 实践启示

1. **先测天花板再决定微调。** 用最强 prompt 加重构任务（尤其检查标签本体是否把正交维度压成一维）跑前沿模型；若像本文卡在 80% 以下、且新一代模型提升微弱，微调才值得做。
2. **别把非专家标注直接当训练集。** 视其为待清洗候选池：先在其上训练，再用模型自评找分歧，只把分歧样本送专家裁定——把昂贵专家时间花在信息量最大的样本上。
3. **多任务用轮转交错，不要完全混合。** 每批一任务、任务间轮转，既保单任务梯度纯度又留共享表示迁移；本文中它贡献最大。
4. **按错误代价设计损失。** 漏检与误报代价不对称时，用非对称裁剪的 CISPO 类损失把容量倾斜到高代价方向，别让对称信任域替你假设两类错误等价。
5. **自蒸馏必须给教师加单调性门控。** 每 N 步提升 checkpoint 为教师，仅当验证指标创新高才升级；否则学生朝被污染的教师收敛。
6. **把 accuracy-per-dollar 写进评估。** 高频低单价任务上成本与准确率常同向，只报准确率会漏掉"领域小模型双赢"这一最常见的部署机会。

## 相关实体

- [[entities/thinking-machines-lab|Thinking Machines Lab]] — Tinker 平台与 OPD 方法的来源
- [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]] — 本文基线所用的 critic-free 算法
- [[entities/on-policy-distillation-vs-offline-distillation-loster|On-Policy Distillation vs Offline Distillation]] — OPD 的方法论对照
- [[concepts/model-distillation-compression|模型蒸馏与压缩]] — 教师-学生范式整体图景
- [[entities/agentic-ai-finance|AI Agent 在金融领域]] — 同类金融任务的智能体化路径
- [[entities/heidi-health-clinical-ai-model-fine-tuning-frontier-parity|Heidi Health 临床微调追平前沿]] — 另一领域"专用微调追平前沿"的平行案例

→ [[raw/articles/learning-to-replicate-expert-judgment-in-financial-tasks|原文存档]]
