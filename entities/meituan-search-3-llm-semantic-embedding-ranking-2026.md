---
title: "美团搜索3.0：LLM 语义表征在排序模型的应用"
created: 2026-08-20
updated: 2026-09-07
type: entity
tags: [meituan, search-ranking, llm-embedding, semantic-representation, cosine-similarity, infonce, triplet-loss, lora, mrle, pepnet, hard-negative, retrieval]
sources: [raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026]
confidence: 0.9
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 美团搜索3.0：LLM 语义表征在排序模型的应用

美团服务零售搜索排序团队在 2025 Q4 至 2026 Q2 用三期迭代把 LLM 语义表征系统性引入精排模型，用 cosine 相似度特征弥补传统文本匹配在长尾语义场景的不足，累计 3 个 Launch Review 全量上线并带来显著订单增量。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

## 一期：验证可行性（64 维余弦特征）

一期用轻量开源基座 + 全参数微调，在词表新增 `<|query|>`、`<|item|>`、`<|qi|>` 三个特殊 Token 作为聚合锚点（平均初始化参考 vocab-expansion），通过三次独立 Forward Pass + Attention Mask 保证 Query/item 表征自包含。取最后一层特殊 Token 位置 hidden state 经两层 MLP 降至 64 维，全量推理 Query/POI Embedding 存 Hive，算 cosine 相似度按 10 个分桶边界（[-0.40, -0.30, -0.18, -0.12, 0.00, 0.10, 0.16, 0.22, 0.30]）离散化，每个分桶配可学习 12 维 Embedding 拼入精排特征。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

线上（20% 流量 14 天）搜索支付订单 +0.20%、服务零售订单 +0.27%，长尾 NDCG@5 +2.21pp、长尾 BadCase@1 -2.96pp。一期同时暴露四个短板：缺商品侧语义、全参微调成本高、点击率目标对排序不全面、三次 Forward Pass 效率低。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

## 二期：Query-POI-Deal 三元表征体系

二期系统性重构表征生产全流程，核心矛盾从「是否匹配」分类转向「哪个更匹配」排序。五个关键升级：①训练数据扩为五元组（Query/Deal 正/POI 正/Deal 难负/POI 难负），难负样本取「同请求同商家曝光未点击」，2766 万条；②Prompt 做减法——精简信息陈述+总结引导优于复杂指令（反直觉，Embedding 的 Prompt 是引导聚合语义而非指令遵循）；③LoRA（r=8/α=32，q_proj+v_proj）取代全参微调；④表征提取改为 nn.Parameter 可学习向量覆写序列末位 + 单次 Forward 五路并行（Last Special Token Embedding 优于 Mean Pooling）；⑤损失从 BCE 改为三组 InfoNCE（Q↔POI/Q↔Deal/POI↔Deal）+ 两组 Triplet（欧氏距离 margin=0.5）共五个损失加权。降维从 MLP 换成 MRL-E（嵌套维度 [1024,512,256,128]，推理截前 128 维）以获得多尺度灵活性。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

二期表征应用升级为双组相似度分桶（Query-POI 与 Query-Deal 各自分桶）+ 零向量缺失边界，底层拼接 + 顶层 LHUC 双融合。工程上改「离线训练样本化 + 线上 KV 读取」，模型体积不增反降。线上（20% 流量 7 天）大盘 UV +0.07%、有效点击 QV +0.13%、QV_CTR +0.10pp；关键洞察是语义表征优化「曝光质量」而非「曝光数量」——某事业部 QV 下降但 QV_CTR 提升 0.24pp。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

## 三期：下挂精排迁移 + 全域交叉统计特征

三期把成熟商家表征迁移到下挂精排（对商家下挂商品排序），遇到的最大工程挑战是**覆盖率而非模型适配**：直接复用二期时 Query 覆盖率仅 81.24%、双覆盖率仅 73.61%（商家精排 Query 偏商家意图词如「SPA」，下挂偏商品意图词如「双人 XX 套餐」），重新按下挂样本分布圈选后升至 98.92%/89.81%。这条「覆盖率验证必须是跨模块迁移第一步」成为标准 checklist。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

三期在表征耦合方式上对比 4 种方案，PEPNet 门控最优（+25bp，vs 底层拼接 +18bp、输出塔前 +8bp）：把 cosine 相似度当门控信号调制其他特征权重，高匹配放大、低匹配抑制，比固定位置拼接更灵活。同时补充四类全域交叉统计特征（user×deal / POI cate3 / user×POI cate3 / query×deal）弥补「个性化×商品」和「Query 意图×商品」建模空白；覆盖率<0.1% 的订单特征被消融移除。线上（10% 流量 7 天）服务零售订单 +0.32%、搜索大盘支付订单 +0.35%，且语义+统计特征叠加效应强于各自单独验证——两者互补而非替代。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

## 独立创新点与业内定位

本工作处于「LLM 文本表征」与「搜索排序特征工程」交叉地带。相对业内（TIGER 生成式检索、ANCE 全局 ANN 难负样本、UNGER 模态平衡、Meta/Apple Music/小红书难负策略）有三点独立创新：①cosine 相似度作为直接排序特征 + 底层/顶层双融合（兼顾特征交叉与信号保真，比推荐场景底层拼接更具可解释性）；②「同请求同商家曝光未点击」难负样本天然绑定搜索上下文（引入后 Q2I-Order-AUC +11.02pp vs Click-AUC +4.85pp）；③「搜索词→商家→商品」三元匹配结构 + 三组 InfoNCE Loss 覆盖所有两两关系，公开文献中较少见。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

后续方向：负例质量（false-negative mask / focal 重加权 / 难度阈值）、离散 Semantic ID、专项下挂表征训练、表征-排序闭环（relevance 蒸馏回灌）。^[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026.md]

## 相关实体

- → [[entities/agent-evaluation-turing-meituan-2026|美团图灵评测方法论]]（同美团技术团队评测方法论，互补）
- → [[entities/taobao-flash-sale-ranking-scaling-up-2026|淘宝闪购爆品团精排]]（电商精排 scaling 实践对照）
- → [[entities/ebay-generative-retrieval-rq-vae-semantic-id-2026-06-30|eBay 生成式检索 Semantic ID]]（离散语义 ID 方向）
- → [[entities/douyin-dme-multimodal-embedding-multimodal-retrieval|抖音多模态 Embedding 检索]]（Embedding 检索侧对照）
- → [[raw/articles/meituan-search-3-llm-semantic-embedding-ranking-2026|原文存档]]
