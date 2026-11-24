---

title: "Figure Helix 2.5 人类动作预训练"
created: 2026-09-24
updated: 2026-11-18
type: entity
tags: ['robotics', 'pretraining', 'embodied-ai']
sources: [raw/articles/figure-helix-index-human-action-pretraining-2026]
confidence: 0.7
vxc: 56
score: v=8/c=7/stars=4
provenance_state: extracted
---

# Figure Helix 2.5 人类动作预训练

## 摘要

2026 年 9 月 17 日，Figure 把 Helix 2.5 带进旧金山湾区 30 个陌生家庭执行收拾客厅、叠毛巾、铺床三项任务，到场后不采新数据、不更新权重、不现场换模型：经 Index 人类数据预训练的模型完整任务成功率 56%，随机初始化对照组仅 9%。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]文章同时并置了两条平行证据链——Dyna-2 用 1000h/1万h/10万h/100万h 四档第一视角人类视频预训练，真机闭环归一化得分依次 20%/28%/45%/53%；亮源新创 Light-O1 把互联网第三视角人类动作预训练推到 1200 亿 tokens（约 10 万小时），适配后预测误差呈幂律下降——共同指向一个问题：人类经验能否成为机器人预训练中可扩展、可预测收益的数据来源。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

## 核心要点

- 对照消融：同架构、同优化、同超参、同下游数据与评估，唯一差异是是否经 Index 预训练，30 个陌生家庭完整任务成功率 56% vs 9%；部分完成不得分，安全原因人工干预整回合记败。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
- 离线规模实验：Index 四档严格嵌套子集（总跨度 8 倍），模型大小与下游训练不变，动作预测损失随规模翻倍规律下降；用小规模拟合外推最大规模，预测误差仅为全范围损失变化的 0.54%。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
- 两条实验不是同一条曲线：Figure 尚未公开 1×/2×/4×/8× 预训练规模各自对应的真实家庭闭环成功率，"规模扩大→闭环成功率连续提升"的曲线并不存在。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
- Dyna-2（8 月）已先把规模效应延伸到真机闭环：四档预训练 + 相同后训练 → 14 个真实任务平均归一化得分 20%/28%/45%/53%；这是归一化表现而非成功率，不能与 56% 横向比较。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
- "零样本"有明确边界：三个任务此前已在其他环境用专门数据适配；零样本只指新家庭、新布局、新物体无需现场重训。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
- Figure 走可控供应链路线：Creator 创作者网络累计收到超 1600 万条视频、支付 1500 万美元，新数据以约 35 分钟/秒涌入，经质量过滤、反作弊、去重、再平衡与文本标注；配套 Nscale 初始 35 亿美元（或扩至 60 亿以上）算力合作，拟部署最高 10 万颗 GPU。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
- 另一条路线押注互联网公开视频：CoMo 学潜在运动表示并生成伪动作，HuRo 把人类视频"机器人化"，[[entities/rhoda-scaling-web-video-pretraining-robots|Rhoda]] 用因果世界模型加逆动力学绕开显式动作标签；亮源新创 Light-O1 将其推到互联网规模并提出"迁移缩放定律"。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

## 深度分析

### 互联网人类视频作为预训练语料的规模定律

行业真正的问题是：人类经验能不能像文本之于大模型那样，成为机器人预训练中一种可持续扩展、且下游收益可预测的数据来源。公开证据已分层落位：最底层是预训练本身随数据增加持续改善；往上一层是改善离开人类数据域后在机器人目标域仍然存在——Figure 的四档 Index 损失曲线与 Light-O1 在动作预测、全身姿态、腕部位置上的幂律误差下降都停在这一层；再往上才是真机闭环——Figure 只证明了"有没有预训练"的 9% 对 56%，Dyna 则给出了目前唯一的规模—闭环单调曲线。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]Dyna 的消融还提示规模效应有条件：单独预测动作时模型随数据增加出现严重而不稳定的过拟合；加入未来视频预测后 39 个机器人任务的迁移整体改善；再补入不带动作标签、只用于视频预测的人类视频，跨本体表现才随规模稳定单调提升。数据是什么、模型被要求从中学什么，共同决定规模效应是否出现。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

### 零样本的定义边界与诚实的实验叙事

"人形机器人零样本做家务"是传播化表述：三个任务此前都已在其他环境用专门数据适配过，零样本仅指首次进入新家庭时不再针对当地布局与物体重训。56% 的口径同样刻意从严——所有玩具必须入篮才算完成，安全人工干预直接记败，这使 9% 对 56% 的六倍差距更有说服力。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]另一处需要克制的推断是数据经济性：Helix 2.5 用约上一代 Helix 02 一半的任务专属数据达到相近成功水平，但这组比较跨越了模型代际与评估条件，不是严格控制的等性能实验；Dyna 用约 10 分钟机器人示范完成 Bottle Cap Untwisting 同理，都不能换算成"多少小时人类数据等于多少小时机器人数据"。两家公司主动用注释说明数字"不是什么"，这种披露方式本身值得行业效仿。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

### Index 预训练 vs 任务专属数据的消融证据

9% 对 56% 是目前该方向上最干净的一组受控证据：架构、优化方式、超参数、下游数据与评估方法全部对齐，唯一变量是是否经 Index 预训练，它回答的是"大规模人类行为预训练是否改变陌生环境中的闭环结果"。四档嵌套实验回答的是另一个问题——预训练规模扩大时，离线迁移收益是否遵循可预测规律，答案是损失随翻倍规律下降且外推误差仅 0.54%。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]Figure 自建供应链的前提写在 Index 发布稿里："要规模化训练真正通用的机器人，所需要的数据并不存在于来源广泛的互联网视频。"因此 Index 不只是数据集，而是一套能主动改变训练数据分布的基础设施：缺什么定向补采，哪种环境过多就再平衡。互联网原生路线（CoMo、HuRo、Rhoda、Light-O1）则反向押注：不重新生产数据，而是提高模型从公开视频提取可迁移信号的能力，把采集成本换成表示恢复、对齐与算力成本。两条路线孰廉孰贵，要算的是从一段经验到可执行能力的整条成本链，而不是原始小时数。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

### 与 Dyna 的时间线与证据链对比

准确的时间线不是"Figure 第一次发现人类到机器人的规模定律"。Dyna-2 在 8 月已经给出更完整的一环：严格嵌套的四档数据、相同的后训练、闭环得分随规模单调上升，并证明了规模效应需要正确的训练目标配合。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]Figure 的增量是把可预测的迁移规模效应明确延伸到全身人形系统，并用 30 个陌生家庭让人类预训练的现实价值变得直观；亮源新创（9 月 21 日技术博客）则把数据源推到离机器人分布最远处——互联网第三视角人类行为视频，37.5 亿到 1200 亿 tokens（约 10 万小时），幂律下降被命名为"迁移缩放定律"，但其证据仍停留在适配后的离线迁移。三条证据链是互补的台阶而非互相否定；三套指标定义互不相容，任何横向比较都需要先对齐口径。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

## 实践启示

1. 评估任何"机器人规模定律"主张时，先确认证据落在哪一层：预训练损失、适配后离线迁移、还是真机闭环——三层结论不可互换，也不能外推。
2. 引用数字时标注口径：56% 是严格完整任务成功率，20%–53% 是平均归一化表现，Light-O1 的幂律是预测误差，三者不可横向排序。
3. 复现 Figure 式消融要完整控制变量：架构、优化、超参、下游数据、评估流程全部对齐，把唯一差异留给预训练本身。
4. "零样本"表述应写明边界（新环境 vs 新任务），Figure 用注释主动纠偏的做法可作为实验叙事的范本。
5. 数据路线的选型是整条链路的账本：采集、表示恢复、对齐、算力加上最终仍难省掉的真实机器人经验，而非单纯比较小时数或价格。
6. 值得跟踪的下一个实验：固定机器人适配方案，让不同预训练规模的模型逐一进入闭环，再改变机器人数据预算画等性能曲线——那将是互联网级人类数据经济性的第一个直接证据。

## 相关实体

- [[entities/rhoda-scaling-web-video-pretraining-robots|Rhoda：互联网视频预训练机器人策略]]
- [[entities/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026|LingBot-VA：视频动作预训练]]
- [[entities/egosuite-open100k-embodied-human-video-data-open-source-2026|EgoSuite：10 万级第一视角人类视频数据集]]
- [[entities/waic-minicpm-1-5b-model-2026|MiniCPM-Robot：开源 VLA 具身模型]]

→ [[raw/articles/figure-helix-index-human-action-pretraining-2026|原文存档]] ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]
