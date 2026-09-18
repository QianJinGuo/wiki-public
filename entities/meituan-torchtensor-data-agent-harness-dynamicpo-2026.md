---
title: "美团 2026 三项学术成果：TorchTensor 数据 Agent 系统设计、DynamicPO 偏好优化坍塌、GeoRA"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [agent-harness, data-agent, multimodal, preference-optimization, dpo, post-training, kdd-cup, meituan, stability, agentic-engineering]
sources: [raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026]
confidence: 0.7
provenance_state: extracted
---

# 美团 2026 三项学术成果：TorchTensor 数据 Agent 系统设计、DynamicPO 偏好优化坍塌、GeoRA

美团技术团队 2026-09-17 的这篇文章汇集了三项近期成果：**GeoRA**（ACL 2026 Outstanding Paper，全球仅 18 篇入选）、《**TorchTensor**》（2026 KDD Cup「Data Agents for Complex Data Analysis」国际竞赛冠军）、**DynamicPO**（DASFAA 2026 Best Paper Award）。三者的共同起点是业务中一个「待解决」的实际问题，而本文最有复用价值的两段内容——数据 Agent 的 Harness 工程与「偏好优化坍塌」现象——恰好都落在 wiki 关注的 Agent Harness 与后训练两条主线上。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

> GeoRA 的完整方法与实验另见 [[entities/geora-geometry-aware-lora-rlvr-meituan-2026|GeoRA 实体]]（同一公众号 2026-08-27 的专文，v×c=72）；本实体只保留其生产落地增量。

## TorchTensor：同模型下比的是系统设计

2026 KDD Cup「Data Agents for Complex Data Analysis」是一个垂直竞赛：所有参赛者必须使用主办方指定的开源模型（有一定智能，但不是最强模型），在大量数据表、非结构化文档、视频中完成数据分析任务——**在模型能力相同的情况下，比拼的是系统设计能力**。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

参赛者（大众点评「问点仔」核心技术架构与算法负责人）把难点拆成三块：**Agent Harness 的设计与稳定性保障**（比赛要求系统在离线环境中无人值守地连续处理数百个任务，稳定性差可能直接导致任务失败）、**视频信息理解**（视频包含大量重复画面，真正有效信息可能只存在于少数关键帧，全部输入会大量占用上下文，还要综合语音与页面结构）、**非结构化文档理解**（简单切分后逐块抽取容易遗漏实体或字段）。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

对应的解题思路是四条工程决策：

- **主动缩小搜索空间**，而不是给模型无限自由，以提高整个系统的稳定性；
- 利用**视频编码信息识别画面变化并筛选关键帧**，通过任务上下文增强语音识别以降低同音字错误率，再把画面、页面布局与语音按时间对齐，让三个信息通道相互验证；
- 对非结构化文档，**先确定完整的实体集合，再按属性分组并行抽取**，通过程序校验与失败重试保证完整性，最后合并、归一化成结构化表格供 Agent 统一查询分析；
- 不急于提交第一版：先花两天研究题目与数据、理解真正难点，再构思整体工具体系与多模态处理方案。

^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

效果：实验中这套定制流程**比强模型（GPT-5.6-Sol）加通用流程的正确率高出 13.3%**；对视频与非结构化文档的理解成为比赛拉开差距的关键因素。开源地址：<https://github.com/zhezh/kddcup2026_champion>。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

## DynamicPO：「偏好优化坍塌」

DynamicPO 由美团业务研发平台商业增值技术部与**中国科学技术大学**共同完成。起点是一个反直觉的实验观察：业界通常认为负样本越多模型越能分清用户偏好，但美团观察到相反现象——**负样本越多，那些「易区分样本」主导了优化，而真正定义偏好边界的「困难样本」却被忽略，于是推荐准确率不升反降**；论文把这一现象称为「偏好优化坍塌」。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

根因在于负样本的构造方式：用户在同一页面看到火锅和拉面却选择了火锅时，未被点击的「相似但没有被用户选择」的样本才真正刻画了「喜欢与不喜欢」的边界；而最常见的 DPO 做法是把一个正样本配上多个负样本一起训练，其中大量是「一眼就不相关」的易区分样本（例如点餐场景里的 4S 店）。此外，负样本比例长期依赖经验试错（1:3、1:5……），在某场景有效的值换到另一场景未必成立。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

解法是**让模型在训练过程中动态筛选更接近偏好边界、信息量更高的负样本，并根据样本难度调整训练力度**。在一组公开数据实验中，当负样本增加到 15 个时，DynamicPO 把推荐命中率从 **58.47% 提升到 66.61%**。团队还把它设计成轻量化、即插即用的模块，可嵌入后训练流程，适用于更多预测用户偏好的场景。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

## GeoRA 的生产落地增量

GeoRA 是首个把 LoRA 与 RLVR 结合起来的方法：RLVR 更新的是稀疏子空间而非 SFT 式的稠密更新，直接在稀疏子空间做稀疏微调会撞上硬件瓶颈（实测单步耗时比全参数还涨 10.8%），因此需要**把 RLVR 的稀疏特性转化为低秩特性**（几何定位 + 低秩压缩）。在真实业务的 Agentic RL 中落地时，GeoRA 仅用**不到全参训练一半的资源**，取得了优于全参和 LoRA 的效果。^[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026.md]

## 资源

- GeoRA 论文：arXiv:2601.09361
- TorchTensor 开源：<https://github.com/zhezh/kddcup2026_champion>
- DynamicPO 论文：<https://arxiv.org/pdf/2605.00327>
- 文末另含「清华-美团学术论坛：物理世界中的 AI 及大模型」（2026-09-21）报名信息与招聘信息，非技术内容。

## 相关

- [[entities/geora-geometry-aware-lora-rlvr-meituan-2026|GeoRA：面向 RLVR 优化几何的低秩适配（ACL 2026 杰出论文）]]
- [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]]
- [[concepts/data-agent-platform-architecture|Data Agent 平台架构]]
- [[concepts/agent-self-improvement-loops|Agent 自我改进循环]]
- [[entities/apo-autonomous-preference-optimization|APO：自主偏好优化]]
- [[entities/localdpo-cvpr2026-video-diffusion-local-preference-taobao|LocalDPO：局部偏好优化]]
- [[entities/rlvr-entropy-collapse-steer-acl-2026-outstanding|RLVR 熵坍塌与 STEER]]
- [[entities/tsinghua-popo-group-prioritized-off-policy-optimization-rlvr|POPO：分组优先离线策略优化]]
- [[entities/aws-sagemaker-sft-dpo-tool-calling|SFT/DPO 工具调用微调]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[moc/reinforcement-learning-rlhf|MOC：强化学习与 RLHF]]
- [[moc/layer-3-agent-engineering|MOC：Agent 工程（Layer 3）]]

→ [[raw/articles/meituan-torchtensor-data-agent-harness-dynamicpo-2026|原文存档]]
