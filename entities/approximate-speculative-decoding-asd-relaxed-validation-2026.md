---
title: "ASD：近似投机解码（Approximate Speculative Decoding）— 预算化最长前缀验证"
created: 2026-09-06
updated: 2026-09-28
type: entity
tags: [inference-optimization, speculative-decoding, llm-engineering, decoding, throughput]
sources: [raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# ASD：近似投机解码（Approximate Speculative Decoding）

> 北航/清华/港大/北大联合团队（arXiv 2608.03447，代码 https://github.com/Kissmetothemoon/ASD）提出。在标准投机解码「首个分歧即截断」规则上引入预算化的近似验证：有选择地接受「大模型本来也几乎想选」的分歧 token，并把其后仍与贪心选择一致的后缀直接复用，免训练、即插即用地提升端到端吞吐。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 问题：标准验证的固有浪费

投机解码（[[concepts/speculative-decoding|Speculative Decoding]]）用轻量草稿模型先猜整块 token，再由大模型一次并行「批改」。标准贪心验证采用二元判断：草稿只要在第一个 token 上与大模型 argmax 不一致，验证立即停止，后面已算好的草稿全部作废——即使大模型已经把整段 logits 都算出来了，这些已付出的计算被白白丢弃。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

关键观察：大模型给后面位置打分时本就假设「前面草稿都成立」。若接受前面那一处小分歧，后面紧跟着的一长串 token 很可能恰好仍是大模型的最优选择——它们本来就已经被算对了。token 级别分歧只是任务质量的「不完美代理信号」，`1776` vs `1,776` vs `\boxed{1776}` 字面不同、答案相同。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 方法：三道闸门控制近似代价

ASD 的核心是「与其在第一个分歧处一刀切，不如在严格可控预算内选择性放行」。当一个草稿 token 与大模型不一致时，先计算其「遗憾值」（regret = 大模型最优选择与草稿选择的概率差），越小说明越无伤大雅。三道闸门：^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

1. **局部遗憾门控（regret gate）**：单个分歧的遗憾值必须足够小，且与「它后面还能挽救多少字」相称。
2. **每块异常次数上限（block cap）**：一个草稿块内最多允许几处分歧，避免单块密集放水。
3. **请求级遗憾预算账本（request-level ledger）**：整段生成中累计接受的偏差总量约束在固定预算内，不让误差随输出长度累积。

三者共同作用使近似被显式量化、可审计。接受一个分歧 token 后，后续草稿字在「包含该分歧的新前缀」下被重新打分，其中一段连续后缀往往仍是大模型的贪心选择——这段后缀直接提交，既不需要额外大模型前向，也不需要新的近似决策，这正是提速的主要来源。预算设为零时严格退化回标准贪心验证。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 实验数字

- Qwen3-14B + DSpark-14B 的 7 个任务：固定负载吞吐平均提升 **7.78%**（区间 3.64%–11.73%），平均每轮接受 token 数从 3.85 → 4.20。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- DSpark / EAGLE3 / Medusa 三种草稿框架共 10 组设置全部正增益（3.05%–15.26%，平均 7.52%），最高提速 15.26%。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- 284B 参数 DeepSeek-V4-Flash（8×H20）：验证端接受率提升约 10%–16%。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- 验证器新增逻辑每输出 token 仅 0.045–0.083ms；免训练、免微调、免额外大模型前向；新增算术复杂度 O(K)。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 工程形态

ASD 是独立、即插即用的验证器模块：不重写草稿模型、不改变投机解码整体流程，只把标准验证中「首个分歧即截断」替换为「预算化的最长前缀选择」，插入现有流水线即可工作——DSpark、EAGLE3、Medusa 等算法均兼容。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 与既有工作对比

- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark]] / [[entities/lmsys-dflash-speculative-decoding-2026-06|dFlash]] / [[entities/eagle-3-speculative-decoding-optimization|EAGLE-3]] / [[entities/deepseek-dspark-v4-speculative-decoding-deepspec|DeepSpec]] 都在设计更优的草稿模型或采样策略；ASD 不换草稿模型，而是放宽验证规则本身，作为它们之上的可叠加验证器层。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- 与 [[concepts/inference-optimization|Inference Optimization]] 家族中 Decode 阶段优化的其他思路（量化、投机采样、paged attention）正交。

## 深度分析

### 三道闸门把「近似」从启发式变成受控预算

标准投机解码的验证规则是布尔式的：分歧出现即截断，没有任何中间态。ASD 把「能否接受这处分歧」拆解成三级递进的控制结构——regret gate 在局部衡量单点代价与「能挽救多少字」的收益是否相称；block cap 限制单块内的分歧密度，防止预算在某一处被集中耗尽；request-level ledger 则在整个请求范围内约束累计偏差总量。三层的价值在于把近似误差从「随输出变长静默累积的隐性风险」变成一个显式、可审计的预算量。消融实验也印证了这一结构设计的必要性：仅有局部放宽的对照方案（MARS 式、Fuzzy 式）在 GSM8K 上分别只有 +5.3% 和 +5.1%，而完整 ASD 达到 +6.5%；MATH-500 上对照为 +6.0% / +7.3%，ASD 为 +9.7%——去掉请求级账本的局部启发式明显更弱，全局约束不是可省略的装饰。

### 提速来源：解锁「已算好后缀」而非降低单次验证开销

ASD 的收益机制容易与「放宽标准换速度」混为一谈，实际核心是 suffix reuse：接受一个低遗憾分歧后，草稿后缀在包含该分歧的新前缀下被重新打分，其中一段连续后缀往往仍与大模型贪心选择一致，可直接提交而无需任何额外前向。这解释了数学类任务收益最高的现象——MATH-500（+11.73%）和 GSM8K（+10.08%）的输出中存在大量格式规范化差异（`1776` vs `1,776` vs `\boxed{1776}`）但推理链高度一致，长后缀可复用的概率天然更大。系统层面的数据进一步排除了「隐藏开销」的解释：验证器自身每 token 仅新增 0.045–0.083ms，而目标验证时间反而因验证轮次减少下降了 1.48–1.51ms，提速真实来自昂贵验证轮次的削减。

### 与标准投机解码构成一个可调的退化谱系

预算 B=0 时 ASD 严格退化为标准贪心验证，这使它不是对现有机制的替代，而是参数化扩展：同一个验证器在「完全无损」与「最大提速」之间提供连续可调的工作点。团队对权衡的处理也相当坦率——明确声明 ASD 界定的是累积局部遗憾的上界，不保证输出逐字一致、语义不变或任务必然正确；在 GSM8K、MATH-500 上超过 95% 的请求输出轨迹发生了变化（哈希分歧），但实测准确率并未下降。这种「吞吐与行为分离审计、取舍显式披露」的评估姿态，比单纯给出平均加速比更值得同类近似方法借鉴。

### 不改草稿改验收：投机解码收益的另一个正交维度

DSpark、EAGLE-3、dFlash、DeepSpec 等主流工作几乎都聚焦草稿生成侧——更好的 drafter、更优的候选树。ASD 的反直觉之处在于证明：验证端本身存在被「首个分歧即截断」规则压制的收益空间，且这部分收益与草稿侧改进天然正交、可叠加——在严格基线已达 1.82×–6.88× 加速的基础上，将上限进一步推到 1.94×–7.32×。同一个验证器挂载到 DSpark / EAGLE3 / Medusa 三种算法、跨 Qwen3 / Llama-3.1 / Qwen2.5 多类目标模型的 10 组设置全部正增益（95% 置信区间均严格大于零），说明收益来自验证端机制本身，与特定草稿算法解耦。对优化路线图而言，这提示了一个此前被系统性忽略的层次：在投入资源训练更强的草稿模型之前，应先评估现有验证规则丢弃了多少已付出的计算。

## 实践启示

- **验证端是投机解码里尚未充分挖掘的优化层**：如果推理服务已采用 [[concepts/speculative-decoding|Speculative Decoding]]，先测量「首个分歧截断」实际丢弃了多少已算好的草稿后缀，再考虑是否值得引入预算化验证器——这可能是比更换草稿模型更便宜的吞吐来源。
- **三道闸门对应三个独立调参旋钮**：预算 B、门控阈值 g、块上限 M 应在独立数据上离线冻结，再针对每个模型与任务组合做质量审计；数学、代码等可校验最终答案的任务，可以考虑把遗憾账本记在任务级信号上，比 token 级 regret 更贴近真实质量。
- **以灰度方式上线，保留零成本回滚路径**：B=0 严格退回标准验证的特性，使其适合作为 [[entities/vllm|vLLM]] / [[entities/sglang|SGLang]] 等框架中的可选验证器模块灰度发布——出现质量问题即把预算归零，无需回滚代码。
- **输出确定性依赖需提前排查**：ASD 会改变绝大多数请求的输出轨迹（>95% 哈希分歧），依赖确定性输出或精确快照对比的下游环节（KV cache 复用、回归测试基线、A/B diff 审计）需先评估影响面。
- **场景边界要认清**：当前 ASD 仅作用于贪心验证；创意写作、开放对话等依赖随机采样的场景（投机采样 + 拒绝采样）尚不能直接套用，需等待采样验证方向的推广。收益最直接的是对话、代码生成、数学推理等对每字延迟敏感的高并发场景。
- **近似优化必须有配套审计流水线**：固定负载测吞吐、自然结束解码单独审计准确率与生成长度的双轨评估，是任何引入近似行为的推理优化落地前的标准动作——没有独立于吞吐指标的质量审计，就不应开启近似开关。

## 相关实体

- [[concepts/speculative-decoding|Speculative Decoding]]
- [[concepts/inference-optimization|Inference Optimization]]
- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark]]
- [[entities/eagle-3-speculative-decoding-optimization|EAGLE-3]]
- [[entities/lmsys-dflash-speculative-decoding-2026-06|dFlash]]

→ [[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026|原文存档]]
