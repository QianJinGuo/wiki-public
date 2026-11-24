---

title: "淘宝生成式检索LLM落地手猫召回"
created: 2026-09-24
updated: 2026-11-16
type: entity
tags: ['retrieval', 'llm', 'search']
sources: [raw/articles/taobao-generative-retrieval-rqvae-cikm-2026]
confidence: 0.7
vxc: 49
score: v=7/c=7/stars=4
provenance_state: synthesized
---

# 淘宝生成式检索LLM落地手猫召回

## 摘要

阿里巴巴（朱建波/玄羿）在 CIKM'2026 发表的 CQ-SID 方案，把 LLM 生成式检索落地为手机天猫搜索的一路召回补充，而非端到端生成。方法上以 RQ-VAE 构建类目约束 + Query-Item 对比学习的语义簇标识符（Semantic ID），用 Qwen2.5-0.5B 经四步渐进式 SFT 建立"文本→个性化 SID"的生成能力，再以专家引导的 GRPO（EG-GRPO）对齐下游排序。离线 Hitrate 显著优于 RQ-VAE 基线（beam@1 +26.76%），线上 GMV+1.15%、UCTCVR+0.40%，已全量部署，生成式召回路的带重曝光占比达 50.25%。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:23-45,295]

## 核心要点

- 手猫既有召回三路各有短板：文本倒排受字符精确匹配限制、存在语义鸿沟；向量检索能捕捉语义相似但难分细粒度属性差异；Trigger 的 I2I 召回相关性有保障但新颖性/多样性不足，易陷入信息茧房。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:23]
- 生成式检索把"索引-检索"流程参数化进单个模型：商品语义编码为离散 SID，LLM 自回归生成候选 SID 列表，检索变成纯生成任务，大幅简化系统架构。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:25]
- 落地定位是关键决策：针对端到端可行性、SID 设计、商品池动态演化、召回-排序一致性四大挑战，本文明确把生成式检索放在召回阶段做补充，而非替代多阶段漏斗。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:31-45]
- CQ-SID 把 SID 视为"语义簇"而非"单商品"标识：三层残差量化（码本 2048×1024×1024），第一级码本用二级类目体系做约束（覆盖 1711 个有效类目），辅以 Query-Item 对比学习。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:56-73]
- 训练走渐进路线：Item2SID → Query2SID → (User+Query)2SID 三段 SFT（Qwen2.5-0.5B），逐步叠加个性化特征；再用 EG-GRPO 在稀疏奖励下向 Group 注入真实 GT 样本稳定学习。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:93-165]
- 资源效率是核心收益：Top-1K 截断下 CQ-SID 只需 Beam Size=30 即达最高 Hitrate=0.4422（基线需 65），Beam 减少 53.85% 的同时 Hitrate 还提升 3.44%。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:214-221]
- 线上采用动态 BeamSize=[20,50,100]，平均总 RT 约 40ms；带重曝光占比 50.25%、点击占比 58.96%、成交占比 72.63%，说明该路召回贡献了高质量增量流量。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:295]

## 深度分析

### 语义簇SID：与"一物一ID"路线的对照

主流方法追求"一物一ID"以压低碰撞率、拉高 Hitrate，但 SID 划分过细会让训练需要更多样本、推理时 Beam Search 需要更大 Beam 才能凑够召回量，RT 与机器资源都会成为瓶颈。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:33] CQ-SID 反其道行之：SID 是语义簇标识，允许语义/属性相似商品挂同一 SID（不追求低碰撞率，挂载量在合理范围即可），同时要求不相似商品映射到不同 SID、相似商品共享 SID 前缀。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:56-62] 实现上基于 RQ-VAE，输入向量复用团队已有的 Query/Item 多模态向量；三层量化中第一级用"商品数大于阈值的二级类目（无则取一级）"做强制类目约束——已知类目的商品直接以类目标签作索引参与码本更新，其余商品走最近邻检索。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:69-75] 训练后对过大簇做随机后处理分割：同 SID 商品本就语义高度相似，随机切分不会破坏语义连贯，却能防止单 SID 挂载过多商品造成检索偏差，并维持前缀层次结构。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:85]

### 四步渐进训练：从Item2SID到个性化UQ2SID

第二阶段建立用户 Query 到 SID 的映射，全部基于 Qwen2.5-0.5B 自回归 SFT（NTP-Loss）：Step1 用商品标题→SID 数据训出基础模型 M1，让模型理解商品文本与各级码本的层次结构；Step2 利用 Query-Item 关联数据，对每个 Query 随机采样 N=3 个关联商品 SID 作目标，得到具备 Query2SID 能力的 M2；Step3 加入用户性别、购买力、近期点击的 Query 相关类目商品 SID 序列等特征，得到个性化模型 M3——同 Query 不同用户兴趣偏好的差异化由此建模。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:93-151] 离线 case 印证了个性化效应：同为 Query"健身器材"，男性用户与女性用户生成的第一级 SID 不同（s1_1388 vs s1_1389），而带历史点击的男性用户则收敛回与其历史行为同簇的 SID。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:271-275] 在线推理时组装 Query 与用户特征，Beam Search 解码 Top-K 个 SID，经预构建的 SID-Item 映射表还原为商品列表，兼得相关性、多样性与个性化。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:169]

### EG-GRPO：稀疏奖励下的召回-排序对齐

前三步 SFT 以点击为目标做 MLE 优化，但好的召回不仅要命中被点击商品，还应给下游排序提供更多可利用的高质量候选。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:37,155] Step4 用 GRPO 继续训练 M3：奖励由查询下用户实际购买、点击、曝光的商品 SID 集合（Ppay/Pclk/Pexp）对生成结果计算。难点在于点击购买远小于曝光、奖励稀疏——实验显示标准 GRPO（K=0）出现模式集中效应：模型把概率堆在少数高置信 SID 上，beam@1 略升但 beam@10 反降、曝光覆盖 pvr 下滑。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:249-258] EG-GRPO 的修法很轻：在每批 Group 采样中从真实点击/曝光的 GT 中随机注入 K 个伪生成结果参与奖励计算与梯度更新，K=2 即带来点击 Hitrate、曝光 Hitrate、曝光覆盖的一致提升，实现精确性与多样性的帕累托改进；K 增大后点击与曝光出现跷跷板效应，但两指标均优于不注入。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:163,258-262]

### 线上落地与资源经济学

两个工程决策值得注意。其一，商品底池筛选：全量商品池会召回大量相关性差或效率低的商品（同 SID 下多为相似品而非相同品，细粒度匹配存在 Gap），因此按效率分筛出约 2100 万商品作为初始底池，每天将新晋高效率商品推理挂载到相应 SID，兼顾时效与质量——这与"召回补充路"的定位一致。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:173] 其二，Beam Size 即资源杠杆：相同 Beam 下 CQ-SID 全面占优（beam@1/10/100 相对基线 +26.76%/+22.57%/+18.89%）；按商品效率分截断 Top-1K 对齐商品量后，CQ-SID 以更小 Beam 达到更高 Hitrate，个性化场景同样成立（Top-1K 下 Beam 减少 17.95%、Hitrate 提升 4.96%）。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:201-208,232-241] 实验还观察到 Hitrate 随 Beam 增大先升后降：离线效率分截断与线上链路有差异，且后续粗排/精排若对大量低质量候选区分能力不足，反而可能把优质商品截断——召回量并非越大越好。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:221] 消融实验证明类目约束与 Query-Item 对比学习各有独立贡献、联合使用产生协同。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:187-208]

## 实践启示

1. 落地生成式检索先定"补一路"还是"端到端"：搜索对准确性要求极高，从召回补充路切入、保留多阶段漏斗，是风险可控的演进路径；端到端留给垂类/长尾场景逐步验证。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:31-45,301]
2. SID 粒度是区分度与聚合性的权衡而非单目标优化：把 SID 当语义簇、以类目体系做第一级先验，能同时压 Beam Size 与保 Hitrate，比盲目追求低碰撞率更划算。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:33,56-73]
3. 个性化应该前移到召回：召回侧输入用户画像与历史行为 SID 序列即可实现"同 Query 不同人不同簇"，与排序侧千人千面对齐。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:131,225-241]
4. 稀疏奖励环境下的 RL 对齐，注入 GT 比调奖励函数更简单有效：EG-GRPO 以 K=2 的小代价同时改善点击、曝光与覆盖，是多目标帕累托改进的实用范式。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:258-262]
5. 用评价口径统一"算力"与"效果"：只比相同 Beam 下的 Hitrate 会因 SID 挂载量不同而失真，补一组按商品量截断的 Top-1K 对比才能回答"同等资源下谁更好"。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:181,210-219]
6. 生成式召回需要配套底池治理：按效率分圈定约 2100 万底池并每日增量更新，弥补语义簇粒度带来的细粒度匹配 Gap。^[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026.md:173]

## 关联

- 同一工作的另一路入库（含线上细节与 RAD-DPO/eBay 对照）：[[entities/llm-generative-retrieval-cq-sid-taobao-search-recall-2026|CQ-SID 电商搜索召回落地]]
- 同为 RQ-VAE 语义 ID 工业实践但取"一物一ID"路线，与本篇语义簇路线互为对照：[[entities/ebay-generative-retrieval-rq-vae-semantic-id-2026-06-30|eBay 生成式检索]]
- 高德侧生成式检索 SID 的三代演进，可对照多场景 SID 设计取舍：[[entities/gaode-saojie-generative-retrieval-sid-three-generations-2026|高德生成式检索 SID 三代演进]]
- GRPO 及其变体的算法脉络：[[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]

→ [[raw/articles/taobao-generative-retrieval-rqvae-cikm-2026|原文存档]]
