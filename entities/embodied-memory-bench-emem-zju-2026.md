---
title: "EmbodiedMemory-Bench（EMem-Bench）：具身智能体长程记忆的四类任务基准与 EMem 三层记忆系统"
created: 2026-10-24
updated: 2026-10-24
type: entity
tags: [embodied-ai, agent-memory, benchmark, evaluation, vlm, long-horizon, memory-system, robotics]
sources: [raw/articles/embodied-memory-bench-emem-zju-2026]
confidence: 0.8
provenance_state: extracted
---

# EmbodiedMemory-Bench（EMem-Bench）：具身智能体长程记忆的四类任务基准与 EMem 三层记忆系统

> 浙江大学 OmniAI 团队（arXiv 2609.28236，项目页 https://zju-omniai.github.io/Embodied-Omni/EmbodiedMemoryBench/，代码 https://github.com/ZJU-OmniAI/Embodied-Omni/tree/main/embodied_memory，数据 https://huggingface.co/datasets/lzLiang/EmbodiedMemoryBench）提出 EmbodiedMemory-Bench（EMem-Bench）：2,554 条可执行交互样本、四类记忆任务，并配套外部记忆系统 Embodied-Memorizer（EMem）与经记忆操作微调的 EMem-8B。基准发布约三周在 HuggingFace 下载接近 30,000 次。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

## 立论：具身任务的瓶颈是记忆，而非感知或规划

团队分析 Gemini-3-Flash 与 Qwen3-VL-32B 在 EmbodiedBench 的 EB-ALF、EB-Hab 上的 100 条失败轨迹，发现 **91%** 的失败与具身记忆相关，并可拆为四类：忽略细粒度视觉线索 **35%**（目标出现过却没记住位置）、遗漏环境状态变化 **18%**（物体已移动仍按旧状态行动）、未记录交互反馈揭示的状态 **31%**（抓取是否成功、环境隐藏状态只能从行动结果确认）、未能迁移过去经验 **7%**（换一个对象又犯同样错误）。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

这四类指向不同的记忆要求：有些信息只在某一帧短暂出现；有些记忆必须随环境变化被更新；有些状态只能通过交互发现；还有些经验需要从多次纠正中归纳并迁移。如「抽屉打不开」无法从单张图像判断，必须依赖一次尝试与反馈；「餐具应放在这个抽屉」则是多次纠正共同表达的规律。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

## 基准设计：四类任务 + 可控干扰轨迹

EMem-Bench 每条样本给出一段按时间排列的多模态交互历史，再给出一个后续任务，考察四类能力：(1) **视觉细节保留**（被动观察中记住短暂出现的线索）；(2) **状态更新**（同一物体多条历史记录时判断当前有效状态）；(3) **交互失败**（记住行动结果揭示的限制，如「上层抽屉锁住」，并据此改用下层）；(4) **经验泛化**（从多个对象的纠正中归纳规律，应用于历史上没出现过的新对象）。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

环境用 AI2-THOR 单房间场景组合成虚拟家庭（离开再返回时房间状态保留）与 ProcTHOR 多房间住宅；规划器在关键线索前后插入**可实际执行、但不触碰关键物体、不泄露答案、不改变任务依赖状态**的干扰交互，把线索与后续任务拉开距离。最终覆盖 **1,118 个场景、125 种可见物体、83 种目标物体、33 种容器及承载物**，自动检查 + 人工审核后保留 2,554 条。前三类任务历史长度中位 **78–83 步**，经验泛化 **111 步**，线索到目标任务的中位间隔仍有 **52–56 步**；被动观察任务的关键画面平均含 **11 个可见物体**。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

## EMem：场景/空间/事件三层互补记忆

外部记忆系统把具身经验拆成三层：**场景记忆**保存选定视觉画面、描述与识别到的物体并绑定所在场景（区分「厨房的杯子」与「卧室的杯子」，相似画面按新增物体与视觉细节筛选）；**空间记忆**用实体图维护物体位置、关系与属性，新位置关系替换冲突旧关系、保留仍然有效的属性，形成「最新已知世界状态」；**事件记忆**按时间记录行动、结果与纠正，保留「这个抽屉打不开」这类交互证据，并可由多次相关纠正归纳成简短规则、关联支持它的原始事件。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

三层记忆形成「观察 → 写入 → 检索 → 行动 → 反馈更新」闭环：模型按时间顺序处理历史，每步依据当前画面、上一动作及其反馈决定写入什么（此时后续任务尚未给出）；接到任务后按场景、实体、关系等结构条件检索、再补文本与语义匹配；动作失败或关键状态变化时中断未执行计划、重新查询与规划。在此基础上对 Qwen3-VL-8B 做监督微调得到 **EMem-8B**，专门学习「存什么 / 查什么 / 如何把检索结果转成行动」三项决策。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

## 实验结果：能力不均衡，记忆系统带来 20+ 个百分点提升

| 设置 | 结果 |
|------|------|
| full-context（完整文本历史 + 当前视觉观察） | Gemini-3-Flash **64.2%** 平均成功率居首；Gemini-2.5-Pro 60.8%；Qwen3.6-27B 58.2%；GPT-5.4 56.8% |
| 同模型跨能力差异 | Qwen3-VL-32B 经验泛化 63.1%，但动态跟踪仅 20.3%；具身专门训练未稳定转化为长程记忆优势 |
| 固定 GPT-5.4-mini 骨干加 EMem | 平均成功率 35.3% → **58.9%（+23.6pp）**；其他记忆系统最佳为 TeleMem 37.8%，EMem 领先 21.1pp |
| 开源骨干无任务训练 | Qwen3-VL-8B / 32B / Mistral-Small-3.1-24B 分别 +10.8 / +14.1 / +16.4pp |
| EMem-8B（含记忆操作训练） | **46.0%**，比原始 Qwen3-VL-8B +20.6pp，比「冻结骨干 + EMem」再 +9.8pp |

消融支持三种记忆的互补性（完整系统优于任一单层）。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

## 工程含义

对长程具身与 Agent 记忆设计的直接含义：(1) 记忆不是单一向量库，「场景 / 空间 / 事件」三类信息需要不同结构（图像绑定、实体图状态、事件-规则）与不同的更新策略（替换冲突状态 vs 累积证据）；(2) 记忆操作本身是可训练技能，微调后的 8B 模型明显优于「冻结骨干 + 外挂记忆」，说明「什么时候写、写什么、查到什么才够」需要专门监督；(3) 评测应把线索与任务之间的干扰步数显式拉长（52–56 步间隔），否则测的是短上下文检索而非长程记忆。^[raw/articles/embodied-memory-bench-emem-zju-2026.md]

## 关联

- 记忆架构：[[concepts/agent-memory-architecture]]、[[entities/ai-agent-memory-systems]]
- 长程任务与具身：[[concepts/long-running-agent-architecture]]、[[concepts/embodied-intelligence-frontier]]
- 记忆评测：[[entities/agent-memory-evaluation-landscape-taobao-survey]]、[[concepts/agent-evaluation-benchmark-frameworks]]

→ [[raw/articles/embodied-memory-bench-emem-zju-2026|原文存档]]
