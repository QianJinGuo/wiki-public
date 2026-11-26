---
title: "Pair Nova 2 Lite with Claude for cost-optimized document processing"
created: 2026-08-01
updated: 2026-11-26
type: entity
tags: ['raw', 'article']
sources: [raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md|原文存档]]

## 摘要

AWS 机器学习博客介绍了一个在 Amazon Bedrock 上构建的两模型文档数字化流水线：Amazon Nova 2 Lite 在单次 Converse 调用中完成原生多模态提取（照片检测与 bounding box、带坐标的姓名提取、页面级元数据），Claude Sonnet 4.6 随后基于页面布局做空间推理，把姓名匹配到对应人脸。在 336 页扫描年鉴上，流水线产出 3,122 条姓名-人脸关联，其中 93.3% 置信度 ≥0.95，仅 0.3% 低于 0.90。两模型拆分方案每页约 $0.033，比单模型全量方案（约 $0.10/页）便宜约三分之二，10 万页可节省约 $6,500 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md]

Nova 2 Lite 采用固定 per-image 计费（每页约 230 image tokens），成本与分辨率无关，便于规模化成本预测；只提取姓名而非全页 OCR，把输出 token 控制在约 1,000/页（全量 OCR 约 4,500）。Claude 通过 adaptive thinking 自动调节推理深度，336 页全部启用了扩展推理，推理轨迹 544–1,658 字符；单页总延迟约 22–35 秒，其中 Claude 空间推理占 20–30 秒。文章还给出批量推理（5 折）、prompt caching（最高省 90%）、reasoning budget 三类进一步降本手段 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md]

## 关键要点

- 两阶段流水线：Nova 2 Lite 单次调用完成照片检测（bounding box，0–1000 坐标系）+ 姓名位置提取 + 页面元数据；Claude Sonnet 4.6 每页调用一次做姓名-人脸空间匹配，两者共享同一坐标系无需转换。
- 实测结果：336 页年鉴 → 3,122 条关联，93.3%（2,912 条）置信度 ≥0.95，202 条在 0.90–0.94，仅 8 条低于 0.90。
- 成本对比：两模型方案 ~$0.033/页（Nova ~$0.0027 + Claude ~$0.030）vs 单模型 Claude 方案 ~$0.10/页；10 万页分别约 $3,300 和 $9,800。
- Nova 2 Lite 按固定 per-image 费率计费（不随分辨率变化），reasoning 设为 LOW 即可——336 页测试中 LOW/MEDIUM/HIGH 准确率无显著差异。
- Claude reasoning token 按输出计价（$15.00/M），Bedrock batch inference 可再打 5 折（Nova 阶段降至 ~$0.0014/页）。

## 深度分析

### 成本结构：省下的三分之二来自"把贵的 token 留给对的任务"

~$0.033/页 的两模型成本里，Nova 2 Lite 提取层只占 ~$0.0027（约 8%），Claude 空间推理占 ~$0.030（约 92%）——推理 token 按 $15.00/M 的输出价计费，是整条流水线真正的成本中心。单模型方案贵到 ~$0.10/页 的原因不是模型能力溢价，而是 token 类别错配：OCR、照片检测、空间匹配全塞进一个 Claude 调用，~6,000 输出 token 全部按 $15/M 计费；拆分后，高容量的结构化提取（姓名 JSON，~1,000 token/页）转移到输出价 $2.50/M 的便宜模型，Claude 只为 ~1,700 个推理 token 付费。省三分之二本质上是一次按 token 单价分流的架构决策 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:260-277]^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:100]

拆分同时买到了两个阶段的独立演进：升级推理模型或更换提取模型互不牵连，而单模型方案是单体 prompt，任何调整都是全局性的。提取质量本身也受益：Nova 原生多模态提取免去了独立 OCR 服务，单模型方案则依赖通用 VLM 的 OCR 能力 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:274-275]

一个反直觉的细节：Nova 每页 ~230 image token 按 $0.30/M 输入价只值 $0.000069——图像输入从来不是这类流水线的成本大头，真正决定每页成本的是输出 token 的数量与单价。对比之下，单模型方案的输入侧反而更重：整页图像按 ~1,500 token 加 ~1,300 prompt token 以 $3/M 的 Claude 输入价计费 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:62-67]^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:270]

### 固定 per-image 计费：可预测性本身就是架构红利

Nova 2 Lite 的图像/文档页输入按固定 per-image 费率计费，与分辨率和文件大小无关。成本函数因此从"分辨率相关、需要 PoC 采样估计"变成"页数的线性函数"：每页 230 image token + ~500 prompt + ~1,000 输出，十万页级工作负载在跑第一页之前就能把预算算准，也不再需要为省钱而维护一条分辨率归一化预处理流水线 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:52-69]。对成本工程而言，这个特性与单价同样重要：可预测的成本才能接入预算告警与 [[concepts/ai-cost-optimization-framework|AI 成本优化框架]] 式的治理流程。

### 置信度分层与失败模式：93.3% 之外看什么

3,122 条关联的置信度分布高度偏斜：2,912 条（93.3%）≥0.95，202 条（6.5%）在 0.90–0.94，仅 8 条（0.3%）<0.90。失败模式与页面类型强相关：282/336 的肖像网格页（平均 10.9 条/页）处理可靠；纯文本页被正确跳过；混合版式页只产出部分关联——肖像匹配成功，说明文字含糊的集体照被放弃而非乱配 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:241-248]

更有用的读法是把置信度当作现成的人工审核路由信号：≥0.95 自动接受、中间带抽检、<0.90 人工复核，人工只需看约 6.8% 的存量。每条关联自带 reasoning 字段解释空间判断（如"第 0 行第 1 位，对应照片上方说明"），坏例可以低成本归因；流水线还为每页输出一张姓名到人脸的连线可视化图，抽检时一眼即可定位错配 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:219-233]^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:252]

### 延迟与推理预算：瓶颈和成本中心重合

单页延迟 ~22–35 秒，其中 Claude 自适应推理占 20–30 秒，Nova 提取仅 2–5 秒——时间瓶颈与成本瓶颈落在同一层。页面相互独立，批量场景可并行摊平吞吐；adaptive thinking 按版式复杂度自动调节推理深度（336 页全部触发扩展推理，轨迹 544–1,658 字符），无需按版式写定制 prompt ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:94]^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:311-317]

成本侧的对应控制面：reasoning token 计入 outputTokens，复杂页会显著推高输出成本，应监控响应元数据里的实际 token 数；简单页可用 budgetTokens 封顶；Nova 侧的结构化提取用 LOW 档推理即可——336 页实测 LOW/MEDIUM/HIGH 准确率无显著差异 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:98-102]^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:42]^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:305]。对能过夜的高吞吐场景，把 JSONL 请求提交到 S3 走 batch 异步处理、次日取回结果即可 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:303]

## 实践启示

1. 按任务难度分层配模型：高容量、模式化的提取交给便宜的原生多模态模型，只在需要空间判断的环节调用贵模型——准确率不降的前提下每页省约三分之二，10 万页节省 ~$6,500；若想让便宜模型承接更难的提取，可参考 [[entities/amazon-nova-lite-fine-tuning-cost-effective-vision-detection-model-tuning-case-and-practice|Nova Lite 微调视觉检测实践]]。
2. 约束输出范围是最直接的降本手段：只提取任务所需字段（仅姓名而非全页 OCR），每页输出从 ~4,500 token 降到 ~1,000 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:44]；输出 schema 越窄，便宜模型越能胜任。该模式适用于任何"视觉元素+关联文本"的文档：历史档案、人员名录、房产列表、商品目录 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:333]。
3. 把模型自报的置信度当审核路由器：高置信自动放行、中间带抽检、低置信人工复核，人工只看 ~6.8% 的存量；reasoning 字段加连线可视化让坏例可归因、可定位。
4. 优先让固定计费的模型承接图像输入：per-image 定价使成本与分辨率解耦，省掉归一化预处理，也让大规模预算在开工前算准——单价与可预测性都是选型指标。
5. 降本手段按顺序叠加：先做架构拆分（省 2/3），再叠 batch inference（5 折，Nova 阶段降至 ~$0.0014/页）、prompt caching（缓存命中最高省 90%）、budgetTokens 封顶 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:303-305]；缓存策略落地细节见 [[entities/amazon-bedrock-claude-prompt-cache-strategy|Bedrock prompt caching 策略]]。
6. 选共享坐标系的模型组合：两阶段都用 0–1000 bbox 坐标，提取结果直接进推理 prompt，无需换算归一 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md:126]——多模型流水线的接缝成本常被低估，同坐标系组合把它压到零，整体分层参考 [[entities/from-pdfs-to-insights-architecting-an-intelligent-document-p|智能文档处理架构]]。

## 来源

- 原文: [[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md|Pair Nova 2 Lite with Claude for cost-optimized document processing]]
