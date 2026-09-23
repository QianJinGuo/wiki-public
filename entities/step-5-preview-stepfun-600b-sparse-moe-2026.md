---
title: "阶跃 Step 5 Preview：600B 稀疏 MoE 杀进全球开源前三"
created: 2026-09-23
updated: 2026-09-23
type: entity
tags: [llm, model, moe, architecture, open-source, stepfun]
sources: [raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三]
confidence: 0.7
provenance_state: extracted
---

# 阶跃 Step 5 Preview：600B 稀疏 MoE 杀进全球开源前三

## 核心事实

阶跃星辰 2026-09-20 发布新一代旗舰基座模型 Step 5 Preview。在 Artificial Analysis Intelligence Index（AA 智能指数，v4.2 版：GPQA Diamond 淘汰、私有测试集权重翻倍到 40%）中取得 44 分，进入全球开源模型前三。对照：Claude Fable 5.1 领跑全球（53 分）、GPT-6 Astra 52.8、Kimi K3 开源权重杀入全球前五^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]。

**性价比**：每百万 token 输入 1 美元、输出 2.7 美元，跑完一道 AA 智能指数任务平均 0.71 美元，在 AA「智能指数 vs 单任务成本」图上创造新的帕累托前沿^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]。

## 架构：Narrow but Deep 的反直觉路线

- **稀疏 MoE**：总参数 600B，激活参数仅 27B，原生支持文本与视觉输入，上下文长度 1M^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]
- **Narrow but Deep**：复杂 Agent 任务存在大量多跳依赖，频繁工具调用带来长 Prefill；Prefill 阶段信息需沿时间和网络深度逐层传递，网络深度不足会限制隐式多跳推理。因此将 Transformer 深度增加至 92 层，为长 Prefill 中的信息传播提供更长路径^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]
- **训练稳定性**：92 层网络撞上数值爆炸、梯度尖峰，对 Hidden State、RMSNorm 和梯度做了专门优化^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]
- **多层面稀疏设计**：Sparse MoE ++ Sparse GQA^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]

## 实测场景（water18 内测）

- **3D 游戏生成**：单文件 HTML + three.js 街机飞行游戏，20 分钟生成上千行可直接玩的代码（自写着色器、粒子系统、结算系统）^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]
- **长程 Agent**：12 份中英文合同对照 10 条法务标准审查——120 个判断全部落格、26 处违规全中，额外标出的 12 处复核后均正确；附件中藏的违规同样抓出^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]
- **Deep Research**：9 份互相冲突的材料先定 7 级证据优先级规则再动手，35 处引用精确到文件行号，主动发现材料间无法解释的数据缺口^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]
- **金融尽调**：B 轮公司数据室尽调备忘录 + 勾稽底稿（1030 个活公式），核出 6 条不符，通过三跳关联（银行流水→供应商法人→deck 团队介绍）挖出未披露关联交易；阶跃自建 FinStepBench 三项评测 + 外部 FrontierFinance（220 道专家题、11543 项评分标准）^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]

## 定位

对准 Coding、软件工程、专业知识和金融场景——需要「长上下文、多轮工具调用和持续执行」的复杂任务，即让模型像员工一样接手项目而非回答问题^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md]。

## 深度分析

### Narrow but Deep：为长 Prefill 多跳推理买深度

600B 总参 / 27B 激活的稀疏度（4.5%激活率）在 2026 年旗舰中并非最大，阶跃的差异化押注在深度而非宽度：Transformer 深度堆到 92 层。直觉是复杂 Agent 任务存在大量多跳依赖，频繁工具调用产生很长的 Prefill，Prefill 阶段的信息须沿时间与网络深度逐层传递——深度不足会限制隐式多跳推理的完成。代价是训练难度：92 层撞上数值爆炸与梯度尖峰，需要针对 Hidden State、RMSNorm 和梯度做专门优化才训得稳^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:180-200]。这与 MTFM 等 推荐/LLM 系统把算力集中于高信息密度位置的思路同构——大不等于强，算力分配结构决定能力结构。

### 稀疏的完整栈：从算法稀疏到系统效率

Step 5 Preview 的稀疏不止 MoE 专家激活一层：Sparse MoE 之上叠加 Sparse GQA——处理 1M 超长上下文时通过稀疏索引只挑相关历史进入注意力计算，即"只看该看的地方"。但算法稀疏不天然等于系统更快：稀疏计算的索引与数据搬运模式可能吃掉理论节省。阶跃用 Block-wise Token Merging 将 Indexer 与 Top-k Selection 成本降 8 倍，并合并相邻 Token 高度重叠的 Top-k 结果以改善碎片化计算的 GPU 利用率^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:202-216]。这条"算法稀疏→系统效率打通"的主线是其 Flash 系列（Step 3.5 Flash 196B/11B、Step 3.7 Flash 198B/11B）效率基因向旗舰级的延伸——激活比例甚至比 Flash 系列更低^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:238-250]。

### 面向 Agent 的数据生产与 RL 训练体系

在数据生产体系中 Agent 不只生成样本，还参与知识探索、任务设计、难度与多样性控制及全流程动态编排；分工明确——专家提供原则和关键判断，Agent 负责动态编排，确定性操作交给代码和工具。系统加入 Anti-hacking 检查防止模型利用评测漏洞"看起来成功"，最终构建出百万级可验证高难任务环境（科学推理/软件工程/机器学习研发/专业工作）^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:218-226]。RL 侧针对 Agent 任务变长做了三方面优化（训推一致性、训练效率、长轨迹学习），并把 Context Compaction 与更细粒度奖励分配纳入训练，使 Agent 在有限上下文中持续推进更长任务^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:228-236]。

### 帕累托前沿定位与体系级竞争

44 分 AA 智能指数（v4.2：私有测试集权重翻倍至 40%，靠背题刷分失效）+ 每百万 token 输入 $1 / 输出 $2.7 + 单任务平均 $0.71，在"智能 vs 成本"图上创造新帕累托前沿：同智能水平没有更便宜的，同价格没有更聪明的^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:20-40]。四项内测（3D 游戏生成、12 份合同 120 判断长程 Agent、9 份冲突材料 Deep Research、B 轮数据室金融尽调）的共同点是"像员工一样接手项目"：长上下文、多轮工具调用、持续执行，且金融尽调实测暴露出未披露关联交易（三跳：银行流水→供应商法人→deck 团队介绍）与"成本按收入倒算"疑点^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:60-140]。体系层面阶跃同时亮出三张牌：前沿基座（Step 5 Preview）、全模态矩阵（StepAudio 3 Realtime 语音交互榜全球第一）、终端量产（装机 4200 万台、日均 2000 万人次，STEPX Neo 通过国标 L3）——"AI+终端"的真实噪声数据飞轮是纯 API 模型公司难以复制的差异化^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:252-290]。

## 实践启示

1. **Agent 时代模型选型看深度与 Prefill 路径**：多跳依赖 + 长 Prefill 的 Agent 负载下，网络深度（92 层）直接约束隐式多跳推理能力——评估基座模型不能只看参数量与激活量，架构拓扑对 Agent 负载的影响是结构性的^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:180-200]。
2. **稀疏设计必须下探到系统层才算数**：Sparse MoE + Sparse GQA + Block-wise Token Merging（Indexer 成本 8×↓）的组合说明，算法层的稀疏收益会被碎片化计算与索引搬运吃掉，落地方案要同时覆盖模型结构与 kernel/调度层^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:202-216]。
3. **效率与智能不是对立面**：Flash 系列（11B 激活）到旗舰（27B 激活、1M 上下文）的连续演进证明效率路线能一路延伸到能力上限，2026 年竞争主轴已从"规模 Scaling"转向"能力 × 效率"综合竞争^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:238-250]。
4. **评测可信度取决于防背题机制**：AA v4.2 淘汰 GPQA Diamond、私有测试集权重翻倍，提示选型时应优先参考抗记忆化设计的私有/动态评测，而非公开榜单刷分^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:32-40]。
5. **Agent 数据体系的三方分工可复用**：专家定原则与关键判断、Agent 做动态编排、确定性操作交给代码工具，再配 Anti-hacking 检查——这套生产体系构建百万级可验证任务环境，是 Agent RL 数据工程的参考架构^[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三.md:218-226]。

## 相关

- 开源模型竞争格局：与 [[entities/kimi-k3-2-8t-params-open-source]] 同为中国开源前沿梯队
- 效率技术脉络：[[entities/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心|JetSpec 解码提速]]（阶跃开源的投机解码）、[[entities/大三本科生一作交出792倍加速的投机解码新答卷deepseek和阶跃星辰双双引用|投机解码新答卷]]（DeepSeek/阶跃双双引用）、[[entities/deepseek-v4-详解1m-上下文背后真正发生了什么|DeepSeek V4 1M 上下文]]
- 架构基础：[[concepts/moe-mixture-of-experts-2025|MoE 混合专家]]、[[concepts/transformer-architecture-2025|Transformer 架构]]、[[entities/llm-inference-pipeline-internals|LLM 推理流水线]]
- 原文存档：[[raw/articles/刚刚阶跃step-5-preview发布一举杀进全球开源前三|原文存档]]
