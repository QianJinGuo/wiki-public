---
title: "KDD Cup 2026 TAAC 赛道：推荐系统的统一架构与 Scaling Law 公开考场"
created: 2026-10-23
updated: 2026-10-23
type: entity
tags: [recommendation, scaling-law, kdd-cup, architecture, industrial-recsys, unified-architecture, feature-crossing, sequence-modeling]
sources: [raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture]
confidence: 0.72
provenance_state: extracted
---

# KDD Cup 2026 TAAC 赛道：推荐系统的统一架构与 Scaling Law 公开考场

腾讯广告算法大赛（TAAC）2026 成为 KDD Cup 2026 官方赛道，题目是一句话：让所有队伍在**同一份数据、同一个基线、同一个延迟约束**下各自交出一套「统一架构」，看谁最有效、且放大后仍然有效。这场比赛的价值不在于奖金（两个各 30 万元人民币的创新奖），而在于它把「推荐系统到底该怎么 scale」从各家闭门实验变成了可横向比较的公开考场——几千支队伍在同一张卷子上答题，答案才能互相比较、经验才能层层叠加。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

## 问题的由来：为什么推荐系统不「自然」scale

大模型吃进去的只有一种东西——一串文字，每个 token 同等对待，所以可以用同一种结构（block）一层层往上叠，想变强就叠得更高更宽。推荐系统吃的是两种性质完全不同的东西：**行为序列**与**特征字段**，行业长期解法是分开处理——一个模块把历史压成一两份兴趣摘要，另一个模块琢磨字段之间如何组合，最后拼起来打分。模型小的时候这种分工既稳又好用，但一旦想往大了做，两个问题同时出现：两个模块形状不同，无法像大模型那样用同一种 block 持续堆叠；历史在第一步就被压成摘要，后面网络再大，被压掉的细节也回不来——**多出来的算力没地方使劲**。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

Meta 在最新一代广告推荐基础模型 GEM 上的判断是：大模型那种 Scaling Law 放到推荐系统上「并不会自然出现」。它的解法是把读历史与琢磨字段交错放进同一个统一框架（Kunlun 架构），把序列建模与特征交叉收进同一个 block——换上之后，Meta 在最大规模预训练上才终于看到类似大模型的 scaling 规律，同等算力增长的收益是上一代架构的 2 倍。由此形成的业界共识是：**先统一，才能 scale**。但「怎么统一」各家答案不一，而且各家在自家数据、自家算力、自家延迟要求下做实验，结论根本无法互相比较。TAAC 2026 用竞赛约束强行把变量对齐，正是要回答这一步。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

## 赛制设计：把「可比性」写进规则

比赛规模：52 个国家和地区、13913 人、5746 支队伍报名（学术赛道仅限在校学生，工业赛道报名单位几乎覆盖国内主要互联网企业，也有 Meta 等海外队伍）。预测目标是微信视频号真实广告日志（脱敏）中「用户点开广告后是否下单/下载/注册」，材料就是行为历史加一堆字段；所有人从同一个起点出发——官方基线 HyFormer 本身已是一个统一架构。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

真正定义这道题的是出题人定的规矩，每一条都在切断「刷分」的捷径：^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

| 约束 | 具体规则 | 封堵的作弊路径 |
|---|---|---|
| 延迟 | 推理超时即不算数，工业赛道卡得更死（线上广告排序是毫秒级） | 靠无上限堆叠换精度 |
| 单模型 | 只许提交一个模型，禁止多模型结果平均，组委会先 AI 审代码再人工复核 | 集成/融合刷分 |
| 时间切分 | 训练集与测试集按时间先后切，两边用户互不重叠，测试样本按一天 24 小时比例抽 | 泄漏未来信息、对老用户过拟合 |
| 易预测剔除 | 好预测的转化类型被剔掉 | 只做简单样本 |
| 规模放大 | 初赛每队仅 1/5 张 GPU、一两百万条数据快速试想法；复赛 7 张卡、数据扩到几千万条 | 小数据过拟合的技巧在放大后失效 |

## 六个方案：统一 block 的不同切法

成绩靠前的方案几乎全部放弃了「各算各的、最后再拼」的老路，但每支队伍下手的位置不同，这些切法本身就是可复用的架构选项。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

- **学术季军 UniTokenFormer**：把「信息切成多少份 token、每份装什么」这一预处理细节当作可单独调节的维度来研究，发现模型规模相当时，**多分 token 给广告侧比多分给用户侧更管用**；顺着这个方向把单个 block 做到约 10.6 亿稠密参数。这是「容量分配优先于单纯做大」的第一个证据。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]
- **工业冠军 QueryFormer**：改的是「从历史里找线索」这一步——模型翻历史之前要先想好这次要找什么。基线做法是把特征拼一拼、过个投影网络生成 query，QueryFormer 先让用户与广告的字段互相交换信息，再让 query 通过注意力去读这些字段，想清楚了再去翻历史。回退这一改动掉分最多，在所有单项改动里排第一。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]
- **工业亚军 UniDot**（作者机构全部是 Meta）：把推荐系统最老的操作——向量内积——请回来，在每一层显式计算 query/key 式内积，并通过一条叫 **FM Highway** 的通道直送最后分类器，不让它在深层被反复冲淡。在所有人卷新结构时，一个老方法换了个位置仍然能打。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]
- **Unified Block 创新奖 MaskRec**：追问「所有 token 放进同一空间后，是不是每个 token 都得跟其他所有 token 交流？」答案是不用——按 token 的角色与来源场景规定谁能直接跟谁交互，再加全局与场景内两级记忆 token 汇总信息。**统一不等于一锅乱炖**。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]
- **工业季军 UniBridge**：念叨的是速度——先围着眼前这条广告把很长的历史压成少量意图 token，再和其他字段一起算，进入核心计算的 token 少而固定，又不耽误两类信息尽早碰面。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]
- **CRAFT**：把维度加到 480 时效果仍在涨，构成「加宽仍然有用」的边界样本——加宽不是绝对无效，前提是有新东西可学。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

## 最干净的一组实验：CoSFormer 的三种放大方式

拿下 **Scaling Law 创新奖**的 CoSFormer 做了一个对照极其干净的实验：同样是「把模型做大」，比较三种路径。第一种是直觉做法——把现有表示直接加宽；第二种是往里加可学习但不带任何新信息的空 token（容量上去了，没喂新东西）；第三种是每加一类 token 就带进一种新的预测信号——新的预训练向量、用户与广告的交叉信息、不同时间跨度的历史摘要，同时让容量跟着长。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

结果非常锋利：直接加宽**从第一步就开始掉分**；空 token 前期小涨后很快回落；只有第三种带着新信息一起扩，效果一路向上，直到新加进来的信息开始重复才慢慢停住。结论因此可以写成一句话：**多出来的容量本身不值钱，值钱的是它拿来处理的那些新信息**。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

这不是孤例：UniTokenFormer 发现「容量分给谁比单纯做大更重要」，腾讯自研 RankUp 研究（同一特征学多组向量表示、引入汇总全局信息的 token、把预训练的用户与广告向量交叉构造新 token）独立得出了相近结论，论文结论直接把这类发现列为本届比赛最重要的发现之一。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

## 可迁移到 LLM/Agent 系统的四条判据

这套从几千份「卷子」里抽出的经验，价值不限于推荐系统——任何「想把能力做大」的系统都适用：^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

1. **可堆叠性先于规模**：先有一个能反复堆叠的同构单元（统一 block），算力才有地方使劲；架构不统一时，加算力等于加在一个漏水的桶里。这与 [[concepts/scaling-laws|Scaling Laws]] 在 LLM 侧的前提条件同构——scaling 不是免费的，它要求结构本身可扩展。
2. **新增容量必须绑定新增信息**：CoSFormer 的三种路径是这条判据最干净的实证。放到 Agent/Harness 语境里，等价于「加更多 context window / 更多工具 / 更多 agent 名额」并不自动变强，只有当新容量承载了新信号（新检索结果、新验证反馈、新执行环境观察）时才有收益。
3. **交互没必要全连接**：MaskRec 的稀疏交互（按角色/场景限定可见范围 + 两级记忆 token）说明统一空间里的信息流通本身是需要设计的成本项，而不是越多越好——这与多 agent 系统里「全连接通信 ≈ 噪音放大」的工程经验一致。
4. **公开同题竞赛 = 知识可比较的唯一方式**：作者用 ImageNet 类比——深度学习那一轮爆发起点不是某家公司的闭门实验，而是 2012 年 AlexNet 在一场公开比赛上打穿错误率。所有人在同一张卷子上答题，经验才能一层层叠上去。

## 局限与可复核材料

作者本人给出了诚实的边界：这离一条能像大模型那样画出曲线的真正 Scaling Law「还差得远」，目前只是一条具体线索——推荐系统不是不能 scale，而是要同时满足「有可堆叠的统一结构」与「有足够多的新信息可学」两个条件。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

可复核材料：两轮比赛的数据样本已公开在 HuggingFace（`TAAC2026`，限非商业研究）；论文附录给出两轮数据中每一类特征的覆盖率统计，官方基线与 starter kit 从数据加载到评测齐全；46 篇方案投稿中有 23 篇被 KDD Cup Workshop 接收，完整论文后续在「腾讯广告技术」发布。这场赛事自 2017 年办到本届已是第七届。^[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture.md]

## 关联

- 同届 KDD Cup 但不同赛道（DataAgents 复杂数据分析）的冠军解法：[[entities/kdd26-美团学术论文精选及kdd-cup26-dataagents赛道冠军思路解读|KDD'26 美团论文精选 + DataAgents 冠军]]
- 推荐系统 scaling law 的另一条硬件侧证据：[[entities/huawei-fuxi-recommendation-system-ascend-npu-scaling-law|华为伏羲推荐系统 Ascend NPU Scaling Law]]
- 推理侧推荐模型（把推荐做成 reasoning 问题）：[[entities/onereason-kuaishou-reasoning-recommender-system|OneReason 快手推理式推荐]]、[[entities/kuaishou-agentx-self-iteration-recsys-2026|快手 AgentX 自迭代推荐]]
- LLM 原生推荐的工业实践：[[entities/genrec-towards-llm-native-recommendation-at-netflix|Netflix GenRec]]
- Scaling Law 的架构/数据/优化三重拆解：[[entities/解构scaling-law优化架构数据的三重奏|解构 Scaling Law 的三重奏]]
- 概念层：[[concepts/scaling-laws|Scaling Laws]]

→ [[raw/articles/kdd-cup-2026-taac-recsys-scaling-law-unified-architecture|原文存档]]
