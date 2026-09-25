---

title: "Cursor如何把一个通用模型，训成顶级编程 Agent"
type: entity
created: 2026-07-04
updated: 2026-09-26
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/cursor如何把一个通用模型训成顶级编程-agent
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Cursor如何把一个通用模型，训成顶级编程 Agent

**来源**: 高可用架构

**发布日期**: 2026-03-31^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


**原文链接**: https://mp.weixin.qq.com/s/q-YQp4LaUNzunupo4cnNhw ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

---

导读：该文章详细解读了Cursor Composer 2模型的技术报告，聚焦于其两阶段训练流程：持续预训练阶段使用Kimi K2.5基础模型增强编码领域知识，并引入多 token 预测以提升推理效率。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

强化学习阶段强调 agentic 软件工程能力，通过 Anyrun 环境和 Firecracker 虚拟机实现安全执行，奖励系统结合任务成功率与非线性长度惩罚，采用 GRPO 变体进行异步训练。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

CursorBench 基准基于真实用户任务设计，包括模糊指令和复杂诊断问题，不断迭代以避免饱和，提供模型在实际开发场景中的可靠评估。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

作者 AVB（@neural_avb），知名 AI 研究解读博主，YouTube“Neural Breakdown”频道主理人。专注拆解前沿 AI 论文，以清晰、教育性方式解读大模型训练、强化学习与软件工程Agent技术。擅长用 AI 工具辅助阅读论文，深受开发者与研究者欢迎。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

上周，Cursor 发布了 Composer 2 。这是他们专为智能体（Agentic）软件工程设计的前沿级 AI 模型。本文将根据其技术报告，深入浅出地解析核心要点：这些模型是如何训练的、强化学习（RL）框架是如何设计的，以及 “CursorBench” 基准测试到底在衡量什么。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

注：Cursor 并未赞助此文，我只是喜欢研读新论文并记录所学。^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


Cursor(@cursor_ai) Composer 2 is now available in Cursor. ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

特别说明： 本文旨在提供教育性科普。我将严格专注于技术部分，拒绝废话，不带节奏，也不会提及发布周期间发生的任何争议。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

以下是根据 Cursor 官方技术报告整理的 Composer 2 诞生过程 👇🏼^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


## 两阶段训练流程

Composer 2 遵循一个两阶段训练流水线，旨在构建深厚的知识储备和执行能力：^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


- 持续预训练（Continued Pretraining, CPT）：
  此阶段侧重于增强模型的潜在编程能力和特定领域知识，确保模型对编程语言、模式和文档有“深度”理解。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

- 大规模强化学习（Large-Scale RL）：
  这是“智能体化”的训练步骤。RL 用于提升模型的端到端表现。AI 在此学习如何推导问题、在终端执行命令，并在长任务中保持一致性。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

## 第一阶段：持续预训练（Continued Pretraining, CPT）

持续预训练是语言模型训练中常见的后期处理步骤。CPT 的目标是让基础语言模型成为特定领域的专家。^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


你向模型输入特定领域（例如“牙科”）的大规模文本数据，并进行“下一个 Token 预测”训练。目标是增强模型在该领域的知识，即使这可能会以降低不相关领域（如“扑克牌”）的性能为代价。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

Composer 2 的 CPT 阶段旨在将基础模型（ Kimi K2.5 ）转化为“编程专家”，之后才会通过强化学习让它学习如何使用工具或与环境交互。 ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

核心观点： CPT 并不是为了训练“能力”，更多是为了增加模型的“领域知识”。^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


### 1. 基础模型选择：Kimi K2.5

Cursor 团队选择 Kimi K2.5 作为 Composer 2 的底座。^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]


- Ki

^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

→ [[raw/articles/cursor如何把一个通用模型训成顶级编程-agent|原文存档]] ^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md]

## 深度分析

### CPT 数据配比与合成策略

Composer 2 的 CPT 分三个子阶段递进：先是 32k 序列长度的代码为主混合数据批量训练，然后扩展到 256k 长上下文以覆盖现代多文件代码库，最后用一轮简短的 SFT 把知识与具体编程任务对齐^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:78-90]。值得注意的缺口是：技术报告未披露具体公开数据集，也未确认是否使用了 Cursor 用户会话数据——这意味着"数据飞轮是否闭环"仍是外部观察者的开放问题^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:88-90]。CPT 引入的 MTP（多 Token 预测）层通过自蒸馏从零训练，去拟合主模型头对未来 Token 的完整逻辑分布，其价值在推理侧兑现为投机解码的"爆发式"生成加速^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:91-106]。把 MTP 放在 CPT 而非 RL 阶段引入，是一个清晰的工程选择：解码头必须在权重尚未剧烈变化时学到稳定的分布预测，RL 之后再加会导致分布漂移、验证失败率上升。

### RL 环境构建与奖励设计

RL 阶段的工程重心完全在环境侧：Anyrun（Rust 平台）+ Firecracker VM 提供每智能体专属隔离执行环境，支持完整开发栈（含浏览器和 GUI），且具备文件系统与内存级快照/分叉能力，可保存轨迹中间状态、失败后回滚重试^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:112-127]。外部网络由 Anygress 代理并强制策略（如剥离敏感请求头），把"不可信代码"的外部影响面压到零^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:128-131]。奖励设计混合三类信号：RLVR 式的最终任务成功、开口向下的非线性长度惩罚（简单任务重罚冗余 Token、复杂任务允许长思考）、以及代码风格/沟通清晰度/工具使用习惯等辅助奖励^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:133-147]。GRPO 变体的两处修改很有信息量：不做组内优势标准差归一化（避免全对轨迹组放大噪声差异），并移除长度标准化项（把长度控制完全交给惩罚曲线），再配合单轮次制度（每个 Prompt 只训一次）防过拟合^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:164-177]。

### 两阶段衔接的工程取舍

CPT 与 RL 的分工被文章概括为"知识 vs 能力"，但真正的衔接难点在三个交界处。其一，超长上下文（256k）必须在 CPT 阶段就绪，否则 RL 阶段的多文件任务会在截断的观测上训练出错误的检索策略^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:80-85]。其二，训练 Harness 与 Cursor IDE 生产环境完全匹配（共享 RPC 工具库、语义搜索等重资源工具外置为接口、支持不停机热更新工具），保证 RL 中学会的行为在部署后不发生 sim-to-real 偏移^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:155-163]。其三，超长任务的上下文溢出靠 Self-Summarization 兜底——模型自我生成先前动作总结并注入下次观测，这让数百次工具调用的轨迹可以持续训练而不丢状态^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:150-153]。整套系统拆为训练/环境/推理/评估四个独立服务的异步架构，与 Minimax、GLM-5 的设计同源，目标是以互不等待换取吞吐最大化^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:180-186]。

这与本地 wiki 中 [[entities/cursor-复盘-harness模型决定能力上限harness-决定生产下限|Harness 决定能力上限的复盘]] 的结论互为印证：模型能力的天花板由训练侧 Harness 决定，而生产下限由部署侧 Harness 兜底^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:155-163]。

### 与开源后训练实践的对照

与开源社区常见的"LoRA 微调 + DPO"路径相比，Composer 2 展示的是完全不同量级的投入：万亿参数 MoE 底座（Kimi K2.5，1.04T 总参/32B 激活）之上的全量 CPT，加上云端 Firecracker 集群级 RL 环境^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:60-74]。开源后训练 rarely 能承担的几件事在这里都是标配：每条轨迹独立 VM 的隔离成本、单轮次制度带来的数据规模需求（每个 Prompt 只用一次意味着海量多样化任务）、以及动态演进的私有基准（CursorBench 随模型变强迭代 v0→v3 以避免榜单饱和）^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:188-208]。但方法论内核是可迁移的：环境快照/回滚、非线性长度惩罚、去掉 GRPO 归一化这些设计，中小规模实验同样适用。作者也指出策略蒸馏（SDPO、ERL 类 RLRF 技术）正流行但报告未提及 Cursor 是否采用，可视为下一个观察点^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:180-186]。

## 实践启示

1. **知识用 CPT，能力用 RL，不要混**：想让模型"知道更多"（领域文档、API、代码模式）走下一个 Token 预测的 CPT；想让模型"做得成"（多步任务、工具调用）走端到端 RL。在 SFT 里塞长任务反而容易教出表面模仿而非真实规划^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:50-62]。
2. **RL 环境先于算法**：Anyrun 的快照/分叉能力（保存轨迹中间态、失败回滚重试）对训练效果的贡献不亚于 GRPO 变体本身。自建 RL 管线时应优先投资环境的可恢复性和观测保真度，而不是急着调超参^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:112-127]。
3. **训练环境必须镜像生产 Harness**：RL 中的工具接口、RPC 协议、外置语义搜索都与 Cursor IDE 一致，否则训出的行为模式上线即失效。做 agent 训练时把"Harness 一致性"当作独立验收项^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:155-163]。
4. **长度控制交给曲线而非归一化**：开口向下的非线性长度惩罚同时惩罚简单任务的啰嗦和复杂任务的浅尝辄止，比 GRPO 的长度标准化更精细。这可直接借用到带验证器的 RL 实验^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:133-147]。
5. **超长轨迹用自我总结续命**：当任务超过上下文窗口时让模型生成动作总结注入后续观测，是低成本扩展有效任务长度的手段，可直接移植到任何 agent 框架的上下文管理中^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:150-153]。
6. **评测基准要随模型进化**：CursorBench 用真实用户任务 + 模糊指令 + 动态版本迭代（v0→v3）对抗饱和。固定公开榜单上的领先对生产场景几乎没有预测力，自建任务集时同理^[raw/articles/cursor如何把一个通用模型训成顶级编程-agent.md:188-208]。

延伸阅读：[[entities/cursor-harness-model-production-floor|Cursor Harness 模型生产底线]]、[[entities/cursor-router-production-model-routing-2026|Cursor Router 生产模型路由]]、[[concepts/harness-engineering-framework|Harness Engineering 框架]]。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

