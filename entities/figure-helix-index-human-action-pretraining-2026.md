---

title: "Figure Helix 2.5 人类动作预训练"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: ['robotics', 'pretraining', 'embodied-ai']
sources: [raw/articles/figure-helix-index-human-action-pretraining-2026]
confidence: 0.7
vxc: 56
score: v=8/c=7/stars=4
provenance_state: extracted
---

# Figure Helix 2.5 人类动作预训练



# 刷视频也能教会机器人干活！1200亿tokens人类动作预训练，误差按幂律下降

关注前沿科技 2026-09-23 11:09 北京

押注互联网里的人类经验

##### 允中 发自 凹非寺  
量子位 | 公众号 QbitAI

9月17日，Figure把Helix 2.5带进了旧金山湾区30个陌生家庭。它要收拾散落在客厅里的玩具、叠毛巾、铺床。机器人到达现场后，不再采集新数据，不更新模型权重，也不根据现场执行结果重新选择模型。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

视频很容易被概括成“人形机器人‘零样本’做家务”。严格来说却并非如此。三个任务此前都在其他环境中用专门数据做过适配。Figure所说的“零样本（zero-shot）”，是指机器人第一次进入这些家庭，面对新的布局和操作对象时，不再针对当地环境和物体重新训练。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

_注：Figure Helix 2.5在30个此前未采集任务训练数据的湾区家庭中执行整理客厅、叠毛巾和铺床任务。这里的“零样本”指机器人面对新的家庭环境、空间布局和操作对象实例时无需现场重新训练，并不意味着模型此前未学习过这些任务；相关任务此前已使用其他环境采集的数据完成适配。来源：Figure_ ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

视频之外，Figure还公布了两组更容易被忽略的实验。

第一组直接比较是否经过Index预训练。

Figure用同一批任务专属数据训练了两套策略模型，架构、优化方式、超参数、下游数据…

## 核心内容



# 刷视频也能教会机器人干活！1200亿tokens人类动作预训练，误差按幂律下降

关注前沿科技 2026-09-23 11:09 北京

押注互联网里的人类经验

##### 允中 发自 凹非寺  
量子位 | 公众号 QbitAI

9月17日，Figure把Helix 2.5带进了旧金山湾区30个陌生家庭。它要收拾散落在客厅里的玩具、叠毛巾、铺床。机器人到达现场后，不再采集新数据，不更新模型权重，也不根据现场执行结果重新选择模型。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

视频很容易被概括成“人形机器人‘零样本’做家务”。严格来说却并非如此。三个任务此前都在其他环境中用专门数据做过适配。Figure所说的“零样本（zero-shot）”，是指机器人第一次进入这些家庭，面对新的布局和操作对象时，不再针对当地环境和物体重新训练。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

_注：Figure Helix 2.5在30个此前未采集任务训练数据的湾区家庭中执行整理客厅、叠毛巾和铺床任务。这里的“零样本”指机器人面对新的家庭环境、空间布局和操作对象实例时无需现场重新训练，并不意味着模型此前未学习过这些任务；相关任务此前已使用其他环境采集的数据完成适配。来源：Figure_ ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

视频之外，Figure还公布了两组更容易被忽略的实验。

第一组直接比较是否经过Index预训练。

Figure用同一批任务专属数据训练了两套策略模型，架构、优化方式、超参数、下游数据和评估方法保持一致。一套随机初始化，另一套从Index预训练模型开始。前者在30个陌生家庭中的完整任务成功率为9%，后者达到**56%** 。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

这个56%要求完整完成任务，部分完成任务的情况不得分。以收拾客厅为例，所有玩具都必须进入篮子；如果出于安全原因需要人工干预，整个执行回合直接记为失败。按照Figure披露的实验协议，两组模型之间的主要差异是——**是否经过Index预训练** 。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

另一组实验转向离线评测，专门考察预训练数据规模。

Figure用Index的四档嵌套数据（nested data）子集训练模型，预训练数据总跨度为8倍；模型大小不变，下游训练方式也保持一致。每个模型再使用相同的任务数据进行微调，比较留出测试集上的机器人动作预测损失。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

随着Index数据规模翻倍，损失规律下降。Figure还用较小规模实验拟合趋势，再预测最大规模模型的测试结果；官方称，预测误差只相当于整个8倍数据范围内损失变化的0.54%。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

这两组实验结果很容易被囫囵理解为“Figure发现了机器人规模定律（Scaling Law）”，但它们本质上是对不同问题的回答。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

第一组实验中的9%到56%说明，有没有大规模的人类行为预训练，会显著影响机器人在陌生环境中的闭环泛化。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

第二组中的四档数据的损失曲线说明，当人类数据预训练规模继续扩大后，这种收益在下游机器人动作预测上呈现出可测量的随规模变化的曲线。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

Figure目前没有公布1倍、2倍、4倍、8倍Index预训练分别对应多少真实家庭成功率。也就是说，**预训练规模持续扩大，对闭环机器人表现会产生怎样的连续变化，这条曲线还没有公开** 。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

_上：Figure在真实机器人闭环评测中，对比了未经Index预训练与经过Index预训练的模型。在其公开的多项未见任务中，Index预训练模型的成功率均明显更高；Figure在正文中将总体结果概括为完整任务成功率从9%提升至56%。这组实验回答的是：“有没有大规模人类行为预训练，会不会影响真实机器人执行结果？”  
下：Figure进一步使用4档严格嵌套的Index数据，将预训练规模从1倍扩大到8倍，并在模型规模、下游训练和评估流程保持一致的情况下，观察到适配后的机器人动作预测验证损失持续下降。这组实验回答的是：“当人类数据预训练规模继续扩大，离线迁移指标是否呈现可预测的规模效应？”  
两组实验并非同一条曲线。Figure目前尚未公开1倍、2倍、4倍、8倍Index预训练规模分别对应的真实机器人闭环成功率，因此不能据此推导“预训练规模扩大后，真实任务成功率按相同规律连续提升”。_ ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

恰恰是这里，让2026年的机器人预训练出现了一个比“Human Data有没有用”更具体的问题：

**人类经验能不能像文本之于大模型那样，成为机器人预训练中一种可以持续扩展、并能预测下游收益的数据来源？** ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

##  Dyna：人类数据的规模效应已经进入真机闭环

Figure并不是第一个观察到“人类数据越多，机器人越受益”的团队。

8月，Dyna Robotics发布Dyna-2。它把第一视角人类视频构成1000小时、1万小时、10万小时和100万小时四个严格嵌套的数据规模。更大的数据集只是在较小集合上继续增加数据，而且各来源比例保持一致，尽量排除数据分布变化对结果的干扰。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

随后，Dyna把四档模型用同一套机器人后训练方法适配到14个真实机器人任务。四档模型的平均归一化得分，依次达到任务可达上限的20%、28%、45%和53%。也就是说，人类视频预训练时的规模差异，经过同样的机器人后训练以后，最终延伸到了真机闭环表现。 ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

_注：Dyna分别使用1000小时、1万小时、10万小时和100万小时第一视角人类视频进行预训练，再将四档模型用相同的机器人后训练方案适配到14个真实机器人任务。随着人类视频预训练规模扩大，14项任务的平均归一化表现依次达到任务可达上限的20%、28%、45%和53%，说明预训练阶段的规模差异在经过相同机器人后训练后，仍然延伸到了真实机器人闭环表现。这里的20%—53%是Dyna为了汇总不同任务原生指标而定义的平均归一化表现，并非任务成功率，也不能与Figure在30个陌生家庭中报告的56%完整任务成功率直接比较。_ ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

因此，更准确的技术时间线不是“Figure第一次发现人类到机器人的规模定律”。Dyna此前已经展示了更完整的跨本体、从预训练规模一路延伸到闭环真机表现的证据；Figure则把可预测的迁移规模效应明确延伸到了全身人形机器人系… ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]

→ [[raw/articles/figure-helix-index-human-action-pretraining-2026|原文存档]] ^[raw/articles/figure-helix-index-human-action-pretraining-2026.md]