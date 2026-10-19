---
title: "LLM 记忆架构三维分类体系（清华《Memory for Large Language Models》综述）"
created: 2026-08-13
updated: 2026-10-20
type: entity
tags: [llm, memory, architecture, taxonomy, kv-cache, ssm, ttt, moe, survey]
sources:
  - raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026
  - raw/articles/tsinghua-memory-for-llms-3d-taxonomy-mozhi-2026
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LLM 记忆架构三维分类体系

清华大学唐杰团队 + NUS + Bosch AI 的综述《Memory for Large Language Models》（20 页 pre-print，用户提供论文原文）提出架构中心的 LLM 记忆三维 taxonomy：**representation（隐式 vs 显式）× update dynamics（离线 vs 在线）× persistence（短期 vs 长期）**，并形式化记忆写入、路由、状态转移与整合的细粒度机制。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md, raw/articles/tsinghua-memory-for-llms-3d-taxonomy-mozhi-2026.md]

## 核心定位与 Scope

**Memory 已从计算的隐式副产品转变为一等架构设计维度**：Titans/端到端 TTT 支持部署期动态参数更新，Engram 等 lookup 架构解耦存储与稠密计算，Nested 多时间尺度更新模糊训练/推理边界，MoE 构成条件参数记忆。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

**关键 Scope 限定**：本文聚焦 **model-level 记忆机制**（在模型架构或推理时动态中实例化的记忆），**排除 agent-level 或 prompt-based 记忆系统**——与库内 agent 级记忆实体（四学派/six 存储学派/系统设计）构成互补维度。Taxonomy 不是刚性分类，而是揭示貌似不同方法间共同结构的概念框架。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 三维分类框架

| 轴 | 判定问题 | 分类 |
|----|---------|------|
| **Representation（表征形式）** | 记忆是否 endowed with 独立、可控的存储/检索接口 | 隐式（计算副产品，无 read/write/lookup 接口）vs 显式（独立存储组件，清晰 read/write 语义 + 可控更新策略） |
| **Update Dynamics（更新机制）** | 记忆何时能被修改 | 离线（仅预训练梯度更新）vs 在线（推理时实时更新） |
| **Persistence（持久性）** | 信息有效作用跨度 | 短期（单会话/局部窗口）vs 长期（跨文档跨对话留存） |

**Explicit 判定的关键洞见**：显式记忆不限于非参数结构——参数化模块也可构成显式记忆。区分标准不是参数化形式，而是**是否被设计并作为记忆运行（清晰 read/write 语义 + 可控更新策略）**。注意即使 KV cache 物化为显式张量，其访问语义由架构固定、缺乏独立寻址/更新控制——仍是隐式记忆。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 隐式记忆（Computation Dynamics）

- **注意力类（KV Cache）**：标准 Transformer、StreamingLLM、Longformer——与原生计算无缝融合，存储随序列线性上涨、会话结束销毁。
- **稀疏/选择性/结构化**：MoBA、NSA、RATTENTION——不改变 KV 存储本质，仅路由/门控筛选 token，属访问优化而非新增存储模块。
- **循环序列记忆（SSM/线性注意力）**：Mamba、RWKV、Kalman 线性注意力、Gated DeltaNet——全部历史压缩为固定尺寸隐藏状态，O(n) 线性复杂度；短板是压缩丢失细粒度细节、长期状态干扰。
- **共同局限**：容量受硬件/窗口限制、无自主读写控制、无法跨会话留存，不适合长期个性化 Agent 场景。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 显式记忆（Addressable and Persistent）

- **参数化外部记忆（TTT 路线）**：Titans、TTT-E2E、In-Place TTT、MEMORYLLM——骨干之外新增可更新参数，推理时梯度/误差信号实时更新，冻结主干规避灾难性遗忘。
- **查表/检索导向**：kNN-LM、Engram、PlugLM、ExplicitLM——哈希槽/可编辑向量库/独立 KV 池，容量无限扩容、与主干算力解耦，RAG 与原生模型融合桥梁。
- **条件参数记忆（MoE）**：Switch Transformer、Mixtral、DeepSeek-MoE——专家子网存领域知识、路由选择性激活；仅在训练阶段更新（离线记忆）。
- **多时间尺度嵌套更新**：Nested Learning——快慢两套参数更新周期，平衡可塑性（学新）与稳定性（不忘旧）。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 5 种细粒度更新规则（Table II，不互斥可组合）

1. **Optimization-based writing**（TTT/Titans 梯度目标更新）——风险：漂移、不稳定、目标不匹配
2. **State-transition updates**（Mamba/Gated DeltaNet/Kalman 递推）——风险：压缩损失、状态干扰
3. **Signal-gated writing/routing**（AMOR/HAM 熵/预测误差门控）——风险：校准、阈值敏感、延迟相关性
4. **Admission/eviction/consolidation**（StreamingLLM/RATTENTION 缓存淘汰压缩）——风险：不可逆损失、检索偏差
5. **Objective-induced/structural updates**（Engram/MoE 离线架构固定）——风险：刚性、目标不匹配、测试时适应性有限

此细化不是第四轴，而是暴露"更新机会出现后记忆如何变化"的机制。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 混合记忆架构（下一代主流）

1. **层间交替注意力+SSM**：Samba、Jamba/Jamba-1.5、Kimi Linear（3:1 配比）、LightTransfer（层替换免重训）——静态固定比例、无动态调度。
2. **自适应混合记忆路由**：AMOR（熵超阈值才启动注意力精炼）、HAM（预测误差大才写入高保真 KV）——算力随输入自适应，但门控阈值靠人工调参。
3. **隐式短时+显式长期双体系**：Titans、Hydra——底层 KV 管本轮上下文、顶层独立参数/哈希存储池跨轮沉淀，实现计算与存储解耦。
4. **多时间尺度时序混合**：TTT-E2E 批量更新 vs Nested Learning 多嵌套循环——记忆层级（token 级/交互批次级/主干冻结）。
5. **多组件模块化设计**：Hydra（状态空间主干+稀疏全局注意力+MoE 路由+双工作区+事实记忆）、Expansion Span（跨度扩展注意力）。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md, raw/articles/tsinghua-memory-for-llms-3d-taxonomy-mozhi-2026.md]

## 显式记忆的结构风险（解读版缺失）

1. **Capacity Growth**：显式记忆容量可扩展但收益递减，需索引/剪枝/压缩。
2. **Interference and Memory Drift**：持久在线存储新旧共存，无规约则干扰/覆盖/放大噪声；**stability-plasticity tradeoff 仍是核心未决张力**。
3. **Optimization and Convergence**：额外目标/更新循环不共享标准预训练优化保证，频繁测试时更新累积偏差。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 评测方法论（四维）

1. **长上下文检索**：NIAH（distractor 找目标事实）、LongBench、RULER/L-Eval（退化曲线，揭示 attention dilution 与 recency bias）
2. **结构化依赖与推理**：SCROLLS、NarrativeQA——跨长文档整合信息；混合/SSM 模型更稳定但牺牲细粒度 token 召回
3. **遗忘/干扰/稳定性**：显式在线记忆的持续更新后遗忘程度
4. **效率权衡**：同召回精度下显存/延迟，区分"真记忆提升"与"单纯扩大上下文窗口"^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 六大开放挑战

统一 LLM 记忆理论（缺通用数学抽象量化容量/压缩损耗/干扰）→ 终身可塑参数记忆（无遗忘持续学习）→ 可解释可控更新规则（可逆可诊断写入）→ 自适应动态记忆调度（智能控制器分配注意力/循环/检索资源）→ 软硬件协同设计（芯片/加速器适配分层记忆）→ 标准化多维评测框架（容量/保真度/持久度/效率统一标准）。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

## 深度分析

### 隐式/显式之辨才是承重墙

三维轴里只有 representation 真正承重，另两轴的意义取决于它。判据不是参数化与否，而是**是否被设计并作为记忆运行**：MoE 参数分区清晰却离线更新，按本文标准只是条件参数记忆；Titans 走参数路线，却因推理时专门规则更新而归于显式。反过来 KV cache 即使物化成张量，访问语义由架构固定、无独立寻址，仍是瞬态计算状态。这一刀切穿了"RAG/长上下文/微调"三选一的叙事——三者混用了持久性轴与表征轴。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

隐式记忆的天花板是结构性的，不是工程量问题：容量受隐藏状态维度与窗口约束、无自主读写控制、会话结束即销毁，三条都不能靠堆显存消除，只能改变失效点位置。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

### 五条更新规则：组合的收益与代价

五条规则不是第四轴，回答的是"更新机会出现后记忆如何变化"。组合有用的前提是信号互补：signal-gated 决定要不要写，optimization-based 决定写多少，admission/eviction 决定留多久——StreamingLLM 的 attention sink 本质就是没有价值模型的准入策略。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

变坏机制也具体：两条规则写同一 substrate 而目标不同，不匹配随更新累积；admission/eviction 的损失不可逆，门控误丢的内容后续优化补不回来。更隐蔽的是阈值通胀——[[concepts/attention-mechanism|注意力]]门控类方法靠人工调参，每叠一层就多一组校准与延迟相关性风险，论文的 risk 栏本身就是组合风险清单。

### KV cache / SSM / TTT：同一设计空间上的三点

三者不是竞争范式，而是在"压缩什么、写到哪里、谁来决定"上取不同坐标。KV 保留精确内容，代价是存储线性上涨、注意力开销随序列平方增长；[[concepts/ssm-attention-sleep-consolidation-cmu-arxiv-2605-26099|SSM]] 把全部历史压进固定尺寸状态换 O(n)，代价是细节丢失与长期状态干扰；TTT 在骨干外另开可更新参数、冻结主干规避灾难性遗忘，容量随模块参数增长，代价是推理时梯度更新的漂移风险与优化保证缺失。边界由坐标决定，不由实现质量决定。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

稀疏注意力因此不算记忆创新：它不改变 KV 存储本质，只筛选哪些 token 参与注意力，属访问优化而非新增存储模块。读 benchmark 时"路由收益"与"记忆收益"必须分开记账。

### 混合架构为何成主流，以及显式记忆的结构性风险

混合胜出是工程原因：三轴正交，可直接叠。Kimi Linear 用 3:1 固定配比拼循环单元与注意力，便宜但静态；AMOR/HAM 的自适应路由则付出人工调阈值的代价。论文明确指出现有混合体都是静态配置、缺动态调度器——这本身是六大开放挑战之一，所以"混合是主流"与"混合尚未被真正设计"同时成立。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

三项风险里最被低估的是**检索偏差**：lookup 系统中陈旧、冗余条目持续偏置检索，在召回导向评测里几乎不可见，通常以"个性化越用越怪"的形态暴露。容量增长是边际收益递减，索引与剪枝属配套成本；stability-plasticity 仍是核心未决张力，多时间尺度只算缓解。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

### 四维评测及其盲区

最有纪律的是第四维：同召回精度下比显存与延迟，区分"真记忆提升"与"单纯扩大窗口"；退化曲线能把 attention dilution 与 recency bias 分离。^[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026.md]

盲区也清楚：NIAH、LongBench 奖励逐字召回，低估了显式记忆该换来的整合与抽象收益；四维全是单次运行，没有跨周持久性测量；只算读侧成本，不算写入开销；也缺矛盾更新的对抗测试。跨架构分数因此只在同一维度内可比——这正是论文把"标准化多维评测"单列为开放问题的原因。

## 实践启示

1. **先判定要不要显式接口，再选技术。** 需求止于单次会话内的长输入推理，隐式加稀疏注意力足够；要求跨会话沉淀，任何窗口尺寸都解决不了，必须引入可寻址存储。
2. **把更新频率当一等超参，而非能力勋章。** 离线/在线是成本风险决策：在线写入买来适应性，付出漂移与累积偏差。上线前先跑"N 次更新后的旧任务保留率"，没有这条曲线就别开在线更新。
3. **组合更新规则控制在两条以内并显式分工。** 一条管准入、一条管写入强度，避免目标不同的规则写同一 substrate；门控阈值要可观测、可回滚。
4. **交付前必做矛盾更新测试。** 只测召回会漏掉显式记忆最典型的失效：旧条目未被覆盖、检索偏向陈旧内容。用"改一条事实、再看后续回答"的样本验证整合路径。
5. **报效率数字必须锁定同等召回**，否则测的是窗口大小而非记忆质量；同时计入写入与整合成本，否则显式方案的账不完整。
6. **混合架构按"静态配比 → 自适应门控"分阶段推进**，先用固定比例拿基线；校准预期时参照 [[concepts/catastrophic-forgetting|灾难性遗忘]]、[[concepts/moe-mixture-of-experts-2025|MoE]] 与 [[concepts/rag-retrieval-augmented-generation|RAG]] 的既有结论。

## 与库内实体关系

- 与 [[entities/context-engineering-three-memory-paradigms|三种 Agent Memory 方案对比实验]]（MSA/D2L/RAG 实证）互补：那是 agent 级记忆方案选型，本文是 model-level 架构分类
- 与 [[entities/agent-memory-four-schools-comparison-2026-07-22|Agent 记忆系统四学派]] / [[entities/agent-memory-storage-six-schools-quantumtransf-debate-frank|六存储学派]] 维度不同：库内为产品/存储级，本文为架构级三维元分类
- 与 [[concepts/context-engineering|上下文工程]] 的记忆维度（L4 记忆系统）衔接：本文提供统一坐标

→ [[raw/articles/memory-for-llms-tsinghua-paper-arxiv-2026|论文原文存档]]
