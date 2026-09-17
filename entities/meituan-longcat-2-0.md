---
title: "美团 LongCat-2.0"
created: 2026-07-01
updated: 2026-09-17
type: entity
tags: [llm, meituan, moe, chinese-hardware, trillion-parameter, long-context, open-source]
sources: [raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026]
confidence: 0.95
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 美团 LongCat-2.0

美团发布的新一代万亿参数大模型，业界首个在五万卡国产算力集群上完成全流程训练与推理的万亿参数模型。总参数 1.6T，平均激活约 48B（动态范围 33B~56B），原生支持 1M 超长上下文。^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

API 平台：https://longcat.chat/platform/product

## 核心参数

| 参数 | 数值 |
|------|------|
| 总参数 | 1.6T |
| 平均激活参数 | ~48B |
| 动态激活范围 | 33B ~ 56B |
| 预训练数据 | 30T+ tokens |
| 上下文长度 | 1M (百万级) |
| 训练算力 | 五万卡国产算力集群 |
| 稳态日吞吐 | 1T+ tokens/day |

^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

## 架构亮点

### LongCat Sparse Attention (LSA)

稀疏注意力机制，将计算量从平方级降至线性级，在 100 万 Token 的超长上下文中依然保持精准的信息定位与理解能力。^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

### 零计算专家 + ScMoE

通过零计算专家实现 token 级动态激活：简单 token 不消耗算力，复杂 token 自动获得更多计算资源。^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

### MOPD 多专家融合

融合三组专家能力：^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

- **Agent Experts**：专攻工具调用与自主纠错
- **Reasoning Experts**：深耕数学与 STEM 推理
- **Interaction Experts**：优化指令遵循与交互体验

推理时由门控网络根据任务类型动态调度最擅长的专家。

## 训练优化

三方面攻克国产算力训练难题：^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

| 维度 | 成果 |
|------|------|
| 稳定性 | 卡间通信异常处理、弹性扩缩卡、自动故障恢复；月均日故障率降低 70%+；硬件故障日均影响从 8h 压至 10min |
| 正确性 | 自研确定性算子、Bitwise 一致性验证、参数检测；关键模块计算精度提升、Reduce 逻辑优化 |
| 效率 | 流水线调度、显存优化、算子级控核；训练 MFU 提升 1.5 倍 |

## 评测结果

### 编程能力

| 评测集 | LongCat-2.0 | 对比模型 |
|---------|-------------|----------|
| SWE-bench Pro | 59.5 | > Gemini 3.1 Pro (54.2), GPT-5.5 (58.6), Claude Opus 4.6 (57.3) |
| SWE-bench Multilingual | 77.3 | ~ Claude Opus 4.6 (77.8) |
| Terminal-Bench 2.1 | 70.8 | - |

### 办公场景复杂任务

| 评测集 | 分数 |
|---------|------|
| RWSearch | 78.8 |
| FORTE | 73.2 |
| BrowseComp | 79.9 |

均达到或接近前沿闭源模型水平。^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

## 市场表现

- 预览版已通过 OpenRouter 和 longcat.ai 面向全球开放
- 趻身 OpenRouter 全球大模型调用量前三
- 月调用量在 Hermes、Claude Code 和 OpenClaw 分列第一、第二、第三
- 成为最受全球 Agent 开发者欢迎的模型之一

^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

## 应用场景示例

- **AI SQL Agent**：业务人员自然语言查询数据，全链路闭环
- **代码库迁移**：分析旧版插件、梳理逻辑、重构新 API 实现
- **完整应用开发**：从一句话描述到可运行产品
- **3D 交互演示**：一句话生成完整 Three.js 3D 演示
- **AI 小说工厂**：多 Agent 协作完成创意写作到商业变现

^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md]

## 与其他模型的关系

- **国产算力特色**：五万卡国产算力集群完成全流程训练
- **MoE 架构**：1.6T 总参数，动态激活约 48B
- **超长上下文**：1M 原生支持，LSA 稀疏注意力机制
- **开源**：对外开源发布

## 相关概念

- **国产算力特色**：五万卡国产算力集群完成全流程训练
- **MoE 架构**：1.6T 总参数，动态激活约 48B
- **超长上下文**：1M 原生支持，LSA 稀疏注意力机制
- **开源**：对外开源发布

→ [[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026|原文存档]]

## 深度分析

### 国产算力全栈：门槛是「稳定」，不是「训得动」

「五万卡国产算力集群完成全流程训练与推理」背后是三条可度量的工程线：稳定性（月均日故障率降低 70% 以上，硬件故障日均影响从 8 小时压至约 10 分钟）、正确性（自研确定性算子、Bitwise 一致性验证、参数检测）、效率（流水线调度、显存优化、算子级控核，训练 MFU 提升 1.5 倍）^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:29-37]。三者合起来说明：万卡级训练的成本函数由「等待与返工」主导，而非峰值算力，弹性扩缩卡与自动故障恢复是一等变量。这套能力源于 2023 年千卡起步、三年攻克算子适配与通信优化 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:29]，属存量积累而非可采购资源，换集群换卡都要重做容错与弹性策略——开源权重与推理代码只是「最后一公里」，训练侧护城河不可搬运。

推理侧同样值得读细：大规模专家并行聚合访存带宽以支撑低延迟解码，零计算专家机制被嵌进专家并行通信流程（使路由到零专家的 token 真正免于传输与计算），再叠加通信／Attention／GEMM 算子优化与权重预取 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:39]。稀疏性只有在通信路径上被「兑现」才能变成延迟收益，否则 MoE 省下的 FLOPs 会以 all-to-all 等待的形式重新付出去。

### 1.6T 总参 / ~48B 激活：取舍落在内存，不在算力

激活率约 3%（≈48B / 1.6T），动态范围 33B~56B，简单 token 走零计算专家近乎不耗算力，复杂 token 自动获得更多资源 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:23]^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:49]。拆开成本结构：总参数决定显存与权重驻留（部署须按万亿参数规划专家并行、分片与权重预取），激活参数决定每 token 计算量——MoE 省的是算力，不省内存，这正是其推理工程重心落在专家并行与显存优化上的原因 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:39]。参见 [[concepts/moe-mixture-of-experts-2025|MoE 混合专家架构]]、[[concepts/inference-optimization|推理系统优化]]。

动态激活还把「平均成本」变成「分布成本」：对按 token 计价的 API 是定价与毛利结构问题；对自建 harness，账单受 prompt 结构影响——重推理集中成一段深思考与打散进多轮交互，单位成本并不相同。且 48B 是路由分布下的均值，线上分布一旦偏移（更多长链推理、更多非常规领域 token），实际单位成本随之偏移。

### LSA 与原生 1M 上下文：边界、代价与对 agent 的意义

LSA 把注意力从平方级降至线性级，官方描述是「不再逐字逐句地看，而是智能筛选关键信息」，在 100 万 token 上下文中保持信息定位与理解 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:47]。该路线的本质是用检索精度换长度可负担性（同类方案对比见 [[entities/msa-sparse-attention-three-kingdoms-huashu|MSA 稀疏注意力方案对比]]，背景见 [[concepts/attention-mechanism|注意力机制]]）：筛选一定会漏，关键是漏什么。对近邻依赖强、结构可预测的输入（代码库、日志、长技术文档）损失通常可控；对全库精确定位类任务，必须用自己的 needle 测试验证，不能以厂商的 1M 声明替代。代价侧至少三项：单位 token 的注意力预算被摊薄、KV cache 在 1M 长度下仍是显存大头、长上下文任务成功率普遍低于短任务。

对 agent ／ harness 的意义正在这里：1M 上下文让「整仓库 + 多轮工具轨迹 + 长期设定」进入单次调用，把记忆问题从检索（RAG）部分转为选择（工作集）——「AI 小说工厂」用长上下文保障百万字级设定一致性即此路径 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:85]；模型侧把 Agent Experts（工具调用与自主纠错）设为专门专家 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:51]，说明 harness 场景是设计目标而非附带用途（延伸阅读：[[concepts/context-window-economics|上下文窗口经济学]]、[[concepts/agent-memory-architecture|Agent 记忆架构]]）。反向成本同样真实：上下文越长，前缀缓存命中、超时重试与成本控制越依赖 harness 自身纪律。

### 评测数字怎么读：口径、scaffold 与可比性

关键数字：SWE-bench Pro 59.5（Gemini 3.1 Pro 54.2、GPT-5.5 58.6、Claude Opus 4.6 57.3）、SWE-bench Multilingual 77.3（Opus 4.6 为 77.8）、Terminal-Bench 2.1 70.8；办公场景 RWSearch 78.8、FORTE 73.2、BrowseComp 79.9 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:59-61]。

四条解读纪律：其一，SWE-bench 类分数对 scaffold 极度敏感，工具集、尝试次数、pass@k、上下文管理策略任一变动即可带来数个百分点，59.5 对 58.6 的差距正落在此噪声带内，「领先」应读作「同一梯队」。其二，对比模型跨代际且未注明是否同口径复测，亦无误差条。其三，团队对 Multilingual 只说「同一水位」，这份保守反而比 Pro 上的领先更可信。其四，办公场景三项无公开对照基线，只能视为自评区间。结论：这些数字足以支撑「进入第一梯队」，不足以支撑排序结论；选型须用 [[concepts/harness-engineering-framework|自己的 harness]] 与私有任务集复测。

### 开源 + API 双轨：路线差异的本质是约束不同

预览版先经 longcat.ai 与 OpenRouter 全球开放，随即跻身 OpenRouter 全球调用量前三，月调用量在 Hermes、Claude Code、OpenClaw 分列第一、第二、第三 ^[raw/articles/meituan-longcat-2-0-trillion-parameter-moe-2026.md:25]。这是典型双轨策略：先用托管 API 抢占 harness 生态的默认档位，再用开源权重换部署自主性；调用量本身成为获客证据，权重则服务于私有化与国产化采购。

放进国产阵营比较，差异是优化目标差异而非架构差异（MoE + 稀疏注意力 + 原生长上下文已是共识）：DeepSeek 更偏架构与推理成本工程（[[entities/deepseek-v4|DeepSeek-V4]]），GLM 强调长上下文与 agentic coding 生态（[[entities/glm-53-how-chinese-labs-keep-stride-with-the-frontier|GLM-5.3]]），Qwen 以尺寸矩阵与开源生态宽度取胜（[[entities/qwen38-max-first-open-weights-release|Qwen 3.8-Max]]）。LongCat 的第一约束是「国产卡上稳定训练、低延迟推理」，因此把供应链可控性做成了产品特性——这决定它在受限场景的不可替代性，而非通用榜单上的名次。同类万亿级开源 MoE 可横向参考 [[entities/thinking-machines-inkling-975b-moe-open-weights-2026|Inkling 975B]] 与 [[entities/tencent-hunyuan-hy4-preview-770b-moe-2026|混元 Hy4 Preview]]（生态背景见 [[concepts/open-source-ai-ecosystem|开源 AI 生态]]）。

## 实践启示

1. **不要用厂商榜单做选型，用自己的 harness 复测。** 固定 scaffold、工具集、尝试次数与 pass@k，把真实任务（仓库迁移、SQL Agent、终端排障）做成 A/B 集对比现有主力模型——SWE-bench 类分差落在 scaffold 噪声带内，复测成本远低于选错模型的返工成本。
2. **按「总参数估显存、激活参数估算力」建模成本。** 1.6T 决定部署形态（专家并行、显存分片、权重预取），≈48B 决定单 token 计算；把 all-to-all 通信与解码延迟列为评估项，稀疏性若不落到通信路径，省下的 FLOPs 会以等待形式还回去。
3. **把 1M 上下文当「少压缩的余量」，不当「要填满的目标」。** 优先做工作集选择与前缀缓存；先用自己领域的全库定位／needle 测试验证 LSA 精度，再决定是否削减 RAG 与压缩策略。
4. **长轨迹 agent 可更激进地保留原始工具输出与终端日志。** 有 1M 可用时把「过早压缩」降级为后备策略，让记忆问题从检索转向选择；同时为 MoE 路由带来的延迟波动设计超时、重试与流式返回。
5. **双轨接入：先用 API 验证，再用开源权重换自主性。** 用 longcat.chat ／ OpenRouter 快速验证价值，确认后再私有化部署或微调；若落地环境是国产卡或受限算力，其开源推理代码与集群适配经验可显著降低从零适配成本，反之在追求极低延迟或最宽生态的通用云场景，把它放进多模型路由的一个档位而非唯一主力。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

