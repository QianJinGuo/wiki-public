---
title: "LayerRecall：选择性长视频视频 DiT 记忆路由（浙大×港大）"
created: 2026-09-07
updated: 2026-11-25
type: entity
tags: [video-diffusion, video-dit, memory, memory-router, long-video, kv-cache, zju, hku, model-architecture, arxiv]
sources: [raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026]
confidence: 0.72
---

# LayerRecall：选择性长视频视频 DiT 记忆路由（浙大×港大）

浙江大学与香港大学团队提出 LayerRecall（arXiv 2608.28460），解决自回归视频生成模型长程记忆失败的问题：模型一段段生成画面并用 KV 缓存保存上下文，为控计算量缓存优先保留最近片段，更早的身份/属性/场景线索被挤出去。LayerRecall 不把整段历史一股脑塞回模型，而是同时回答两个问题——**现在该想起什么？这段记忆又该在哪些层里使用？**^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

## 两把钥匙：What to Retrieve + Where to Use

- **What to Retrieve（取什么）**：系统为历史 chunk 建立紧凑摘要作检索索引；当前画面到来后，路由器读取当前隐藏状态、形成随生成内容变化的查询，从历史候选中找出最相关记录。技术关键：摘要只负责检索打分，真正送入注意力计算的仍是被选中 chunk 的完整 K/V。
- **Where to Use（用在哪层）**：被选中的「记忆敏感层」同时看到 sink、检索历史和当前 chunk；其他层继续用原始局部滑动窗口。注意：当前状态动态控制「取什么」，「在哪里用」是预先分析并固定的骨干特定策略——不是每一步动态选择网络层。

## 反直觉发现：记忆不是越多越好，更不能每层都塞

逐层分析 LongLive-2.0 发现，有些层更依赖当前/近期画面（维持姿态、运动、局部连续性），另一些层才对远期历史投入更多注意力；且「哪一层需要长时记忆」随骨干不同而异，不能把同一组层位置当通用答案。把历史 K/V 注入所有层的 All-Layer Routing 对照显示，多了历史反而更容易出现突变和时间抖动（高频帧变化功率比 0.60→0.38）。^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

## 训练：Cross-Horizon Prediction Matching（CHPM）

现实中没有人逐帧标注「此刻应找回第几个 chunk」。CHPM 用同一冻结视频骨干构造「看得更远的老师」（可见 384 latent frames）与「记忆受限的学生」（单次注意力预算 32 帧）。两者面对相同噪声/文本条件/扩散时间步去噪预测差异驱动路由器学习；不要求 student 复制 teacher 注意力图。只优化 ~165 万参数记忆路由模块，5B 生成骨干/文本编码器/VAE 均冻结。随机初始化 MemoBench 0.519 → CHPM 训练后 0.548。^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

## 评测

MemoBench Overall 0.548（优于最佳基线 MemFlow 0.531）、MovieBench Overall 0.578（优于 Rolling Forcing 0.548）、VBench-Long 0.978（与 LongLive-2.0 骨干持平）。额外路由代价小：端到端生成 305.9s→309.4s（+3.5s），解码吞吐 5.22→5.16 FPS。^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

> [!note] 边界：LayerRecall 并不显式检测「哪里生成错了」——展示的是与机制一致的行为案例，不是已完成因果验证的纠错模块。

## 深度分析

### 索引与消费分离：摘要只做打分，完整 K/V 才进注意力

LayerRecall 检索链路里有一条容易被忽略的分层边界：chunk 摘要只承担检索打分职能，真正送入注意力计算的永远是被选中 chunk 的完整 K/V^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:76-80]。这与成熟 RAG 系统的「索引用轻量表示、消费用原始载荷」是同一模式：压缩表示负责找，未压缩内容负责用；一旦把摘要直接交给生成端，等于强迫模型用有损副本作画，召回再准也会在消费端漏细节。另一处细节是查询来源——路由器从当前隐藏状态构造查询，「此刻想找什么」由生成状态而非仅由文本提示决定^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:78]。这与 [[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026|MemAgent：记忆重构而非检索]]构成对照：两者都拒绝无差别回灌历史，一个在检索入口做选择，一个在生成端做重构。

### 层分工是骨干「指纹」，路由函数与层策略可以解耦

对 LongLive-2.0 的逐层归因显示：部分层依赖当前与近期画面以维持姿态、运动与局部连续性，另一些层才把注意力预算投向远期历史；且记忆敏感层的峰值位置在 LongLive、Self-Forcing 两个骨干上并不重合^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:62-66]。「哪些层吃远期信息」因此是骨干特定属性，不是可跨架构复用的常数。零再训练迁移实验恰好印证了这套解耦：路由器参数原样搬走，只替换成目标骨干自己的记忆层 profile^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:124-128]。可迁移的是「怎么选记忆」这个函数，不可迁移的是「送到哪几层」这张表——这比迁移分数本身更有设计价值，也呼应 [[concepts/attention-mechanism|注意力机制]]中层间功能特化的一般观察。

### 时机比容量重要：全层注入反而放大抖动

All-Layer Routing 对照给出一个负结果：把历史 K/V 注入所有层后，模型有了更多历史，却出现更多突变与时间抖动^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:68]；频谱上 2–12 Hz 高频帧变化功率比从 0.60 降到 0.38 的改善，正是相对这一全层基线测得的^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:116-118]。合理的解释是：负责短期连续性的层被陈旧信号污染了当前帧统计——历史信息的「注入位置」错了，量越大越有害。对这条与 [[entities/ard-agentic-autoregressive-diffusion-for-long-video-consistency|ARD：自回归扩散长视频一致性]]同属的问题域而言，记忆调度的第一性原则是「合适的内容、合适的时机、合适的层」，而非单纯扩容。

### CHPM：用预测差当零标注监督信号

记忆路由的训练难点在于不存在「此刻应找回第几个 chunk」的逐帧标注^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:94]。CHPM 的做法是绕开标注：在同一冻结骨干上构造可见 384 latent frames 的老师与单次 32 帧预算的学生，在相同噪声、文本条件与扩散时间步下比较两者去噪预测，以预测差驱动路由器学习——既不要求复制老师的注意力图，也不需要知道老师实际用了哪段历史^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:96-100]。这是「对齐结果、不模仿过程」的蒸馏取向（参见 [[concepts/model-distillation-compression|模型蒸馏与压缩]]）：过程模仿需要可信的过程标签，结果对齐只要可复现的前向。代价侧也便宜：随机初始化 0.519 → CHPM 后 0.548，只训练约 165 万参数、权重 6.6MB、端到端生成多花约 3.5s^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:102]。

### 自修复行为的机制叙事与证据等级

项目主页的局部属性修复案例（人物离场返回后内搭颜色一度漂移，随当前镜头变清晰而恢复历史外观）给出的机制叙事是：当前状态越明确，查询越容易对齐到正确历史，记忆敏感层负责修复局部属性，其余层继续维护运动^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:108-114]。但作者自己划了边界：并不存在显式的错误检测回路，展示的是与机制一致的行为案例，不是经因果验证的纠错模块。读这类论文时值得把演示与协议内结果分开记账——MemoBench 0.548 优于最佳基线 0.531 是 100 条三镜头 prompt 协议内的总体结果，个案恢复不能当作纠错能力的证明^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:44-52]。

## 实践启示

1. **检索-消费分离先行**：给任何系统（KV 缓存、RAG 知识库、agent 记忆）加外挂记忆时，先守住「轻量索引负责打分、命中后加载完整原始内容」这条边界；不要让摘要或压缩表示直接进入消费端。
2. **注入位置先于注入数量**：扩容或回灌历史之前，先做逐层/逐模块归因，确定哪些位置真正消费远期信息——位置错了，历史反而放大时序抖动。这与 [[entities/admem-memory-policy-selective-memory-sjtu-tencent-2026|AdMem 的选择性记忆策略]]同属「少而准」路线。
3. **位置策略按骨干重新标定**：记忆敏感层的峰值位置随骨干变化，不能照搬他人发布的层号；把「路由函数」与「层策略表」设计成两个可独立替换的部件，才有低成本换底座的可能^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:124-128]。
4. **没有标注就用预测差**：当记忆决策缺乏真值时，「同一骨干、不同可见窗口的生成预测差」是可自产的监督信号；只对齐输出结果、不模仿内部注意力，省掉过程标注^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:94-100]。
5. **外挂模块做小并冻结宿主**：0.033% 参数占比、6.6MB 权重、+3.5s 开销的代价结构说明路由层可以便宜到随手迁移；宿主冻结还让「外挂失效可整体卸载」成为兜底方案^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md:132-138]。
6. **区分个案演示与协议内结果**：评估任何记忆增强方法时，把「机制一致的行为案例」与「基准总体提升」分开记账；前者用于理解机制，后者才用于技术选型决策。

→ [[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026|原文存档]]