---
title: "高德扫街榜生成式召回三代演进：从兴趣空间到地理空间（GeoGR → HF-SID → SPAR）"
created: 2026-09-17
updated: 2026-09-17
type: entity
tags: [generative-retrieval, semantic-id, lbs, poi-recommendation, geogr, hf-sid, spar, spatial-modeling, gaode, amap]
sources: [raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026]
confidence: 0.9
provenance_state: extracted
---

# 高德扫街榜生成式召回三代演进（GeoGR → HF-SID → SPAR）

高德技术（信息业务中心，2026-09-17）扫街榜（LBS 本地生活推荐）生成式召回三代体系：核心命题是**从"兴趣空间"走向"地理空间"**——主流生成式召回 SID 由文本语义导出，模型学到的是行为共现兴趣空间（知道哪些地方常被一起去，不知道位置关系）；LBS 场景不只"想去"更要"方便到达"，错位直接反映在 SID 上：坐标相邻的 POI 拿不到相邻 SID、连锁品牌相隔几十公里的门店拿到几乎相同 SID，漏掉大量"近且相关"候选。^[raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026.md]

## 三代技术演进表

| 代际 | 核心瓶颈 | 核心方法 | 线上效果 |
|---|---|---|---|
| GeoGR（arXiv:2602.10411） | 通用 SID 缺乏时空理解 | 地理约束共访对比学习+RQ-Kmeans+EM 式 SID 优化+CPT/SFT | vs 传统召回 PV_CTR +3% |
| HF-SID（arXiv:2608.30479） | 无法高精度表征经纬度/数值/结构 | 连续数值编码+三维笛卡尔坐标+Geo-CPT/Num-CPT+结构化对比学习 | vs GeoGR PV_CTR +2% |
| SPAR（arXiv:2609.02062） | 直线距离≠路网真实可达距离 | SI-SID 空间内化+25 数据集 MG-CPT+TV-SFT 防遗忘 | vs HF-SID PV_CTR +2% |

三代共享「POI 表征与 SID 构建 → CPT → SFT → 前缀树约束解码」范式，地理感知层次持续演进：行为中间接学习空间邻近性 → 表征层精确表示坐标/数值/结构 → 点级位置到城市路网可达性理解。^[raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026.md]

## 第一代 GeoGR：地理约束共访对比学习

三步构建时空感知 SID：①时空协同表征——行程动作挖掘共现 POI 对作正样本，**空间邻近性经由「谁和谁常被一起访问」的行为证据进入表征空间**（机场与停车场、景区与酒店的跨类目强关联是文本相似捕捉不到的；避开邻居序列拼输入文本——过长削弱目标 POI 语义）；②三层 RQ-Kmeans 逐层量化残差生成由粗到细 SID；③EM 式优化（固定 SID 微调模型→固定模型按高置信预测更新 SID，迭代对齐离散结构与生成分布）。CPT 对齐 SID token 与基座+注入知识；SFT 输入位置/查询/时间/天气/动作特征输出三层 SID。数据：同 SID 平均地理距离 **6.07km→0.49km**、语义相似度 0.62/0.73/0.90→0.82/0.98/0.99（地理约束未牺牲语义一致性）；消融共访对比学习 Recall@5/NDCG@5 +30.00%/+33.56%、EM +16.40%/+24.78%、结合 +37.56%/+37.48%、再加 CPT Recall@5 0.4732→0.5068；GeoGR-4B vs OneLoc 全国 AMAP +12.4%/上海 +14.6%（工业数据越大价值越明显）；部署 0.6B（50ms；4B 90ms 太慢），三个月 A/B **PV_CTR +3%/PV_CVR +4%**，生活服务场景最敏感 +9.37%。能力边界：地理监督是定性判断（阈值内实际距离不同被当作同类正样本），无连续坐标保真。^[raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026.md]

## 第二代 HF-SID：坐标与数值的连续编码

三类表征困难：坐标缺乏连续性（数字子词/Geohash-S2 网格边界突变）、数值缺乏量纲区分（评分 0-5 与访问量 0-12 万共享子空间，"差 0.5"意义完全不同）、结构读不出来（Apple Store vs Apple Market 细类目不同；商圈内无关 POI 被收进同一前缀）。原则：**先在连续表征空间补齐保真度，再执行离散量化**。方法：经纬度换算米制地心三维坐标（欧氏距离与球面距离严格单调、城市尺度近似线性、消除 ±180° 跳变）；数值拆符号+九个整数位+小数并平滑进位跳变，MLP 升维后用判别符号/逐位分类整数位/回归小数三任务预训练（迫使量级结构沿独立方向线性可读），编码器冻结；**type embedding 按属性类型区分**（作用类似位置编码）；每个数值只占一个 [NUM] token（token 数不随数字长度膨胀）；Geo-CPT（两点距离计算+最近 POI 对识别）+Num-CPT（同类型属性跨 POI 比较）混合监督。结构感知：前两层直接量化，末层量化前用两级类目标签对第二阶残差对比学习精修（粗同细不同=困难负样本），只更新末层投影 SID 仍三 token。数据：簇内平均距离 **0.49→0.25km**（降幅 95.9%），Hit@200 81.24%（+3.66pp）；**地理紧凑与码字区分兼得**（RQ-OPQ 独立率 98.93% 但 7.27km；GeoGR 0.49km 但独立率 54.69%；HF-SID 0.25km+84.22%）；市级 NMI 0.2615→0.9796；线上 **PV_CTR +2%/PV_CVR +3%**。能力边界：直线距离≠真实可达距离。^[raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026.md]

## 第三代 SPAR：城市路网知识注入+防遗忘双分支微调

三问题：直线近但河流/高架/单行道需大幅绕行；行为共现兴趣空间训练（预训练语料覆盖不到城市细粒度 POI 分布与路网）；CPT 注入空间知识后被海量行为 SFT 遗忘。「构建—培育—保持」：**SI-SID** 借鉴 NeRF/位置编码——经纬度归一化到周期区间投影到指数递减频率取正弦余弦，两层 MLP 得地理向量，与文本语义拼接 L2 归一化后三层 RQ-Kmeans+消歧索引=四 token（具有位移不变性与 Lipschitz 连续性）；**MG-CPT** 基于真实 POI/道路/导航数据构建 25 个空间数据集三层级（基础属性→空间关系→系统性导航知识：路线距离/通行时间/有序道路序列建模路网连通）；**TV-SFT 双分支低秩微调**——MG-CPT 相对基座的参数变化差=「城市空间知识」任务向量整体冻结，行为分支对基座全参微调拟合兴趣空间，空间分支只外挂 LoRA 增量（融合系数 1=在 MG-CPT 上微调；0=退化普通全参 SFT），空间知识被锚定不被行为数据冲淡。数据：消融 SI-SID +10.76%/MG-CPT 再 +15.38%/TV-SFT 再 +2.15-17.90%；**空间认知来自定向训练非参数量**：Qwen4B 经 MG-CPT 在 18 类空间认知任务 35.9→78.1，超 8 倍参数的 Qwen32B（44.3）达 34.8pp（通用 scaling 只提升 8.4pp）；无坐标时方向判断仍 76-79 分（学到城市空间布局本身而非拿到坐标现算）；navi_ 导航任务 4B 显著优于 32B；线上 **PV_CTR +2%/PV_CVR +4%**。^[raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026.md]

## 与既有实体的关系

- `[[entities/ebay-generative-retrieval-rq-vae-semantic-id-2026-06-30|eBay 生成式检索 RQ-VAE 语义 ID]]`：同生成式检索技术族（RQ-Kmeans/RQ-VAE 量化 SID），eBay 是电商文本语义维度，本文是 **LBS 地理空间维度**（兴趣空间→地理空间）——该维度在 eBay 实体零覆盖 → SUPP 补强
- `[[entities/gaode-saojie-image-selection-hermesagent-vlm-production-2026|高德扫街榜图像选择]]`：同扫街榜业务系列（VLM 图像选择侧 vs 召回侧）
- `[[entities/harness-engineering|Harness Engineering]]`：非直接相关，SID 三代演进属推荐系统工程

→ [[raw/articles/gaode-saojie-generative-retrieval-sid-three-generations-2026|原文存档]]
