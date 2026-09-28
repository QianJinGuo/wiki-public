---
title: "Speculative Programmatic Tool Calling (sPTC) — Alex Zhang 2026"
created: 2026-08-26
updated: 2026-09-28
type: entity
tags: [harness, inference-optimization, rlm, tool-calling, speculative-execution, code-execution, latency]
sources:
  - raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Speculative Programmatic Tool Calling (sPTC) — Alex Zhang 2026

## 核心贡献

**Speculative Programmatic Tool Calling (sPTC)** 是一类将工具调用计算与 harness 正在生成的代码进行重叠（overlap）的技术，灵感来自 CPU 的 speculative execution 和 LLM 的 speculative decoding。核心技巧：在 harness 仍在生成 token 时，从**部分生成的 REPL 调用**中推断并预启动工具调用，而不是等整个生成完成。若最终生成的 REPL 确实调用了这些工具，它们立即从预启动调用的缓存输出返回。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

关键观察：LLM 工具（sub-agent、search API）通常高延迟，是依赖代码执行作为动作的 harness 的瓶颈；同时主上下文生成本身也常是阻塞中间调用的显著延迟瓶颈。sPTC 的两个主要节省来源：(1) 在 token streaming 期间重叠已生成的工具调用；(2) 作为 REPL 调用之上的"朴素 JIT 编译器"，把代码中未写成异步但实际不阻塞的独立调用并行化。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

## 设计：Shadowed Execution

高层设计是让 REPL 中导入的工具调用带一个可提前调用、在需要时替换为缓存输出的 hook。契约定义：(1) 哪些工具应被 speculate（如 sub-LLM 调用可以，sub-RLM 调用成本太高）；(2) 对输入依赖先前内存变量的工具调用的 speculate 机制。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

核心机制是 **shadow REPL**：主代码 REPL 的 deepcopy fork，实时执行部分 REPL。大多数外部库和不安全函数（如 `open`）标记为 "unsafe"，任何输入依赖这些函数的 speculatable 工具都不 speculate。speculator executor 故意不用作真实 REPL executor——因为模型产生的完整 REPL 可能含错误代码或不完整工具调用，此时不希望 speculator 修改 REPL 状态，把整个 REPL cell 视为计算单元。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

可 speculate 的情况分四类：
- **Case 1 Literals**：字符串/整数字面量可立即解析为工具调用，无需 shadowed execution
- **Case 2 Input dependencies**：所有输入 "safe"（纯函数、无副作用）即可 speculate；依赖被 speculate 的工具会先等依赖计算，即使 LLM 仍在 streaming 也执行
- **Case 3 Peekable 依赖**：streaming 中 shadow REPL 的工作命名空间使即使依赖是内存变量也能 speculate
- **Case 4 Blocked**：allowlist 定义可用来计算输入依赖的关键词/函数；被阻塞的工具不 speculate

在 PTC 中，工具调用按其输入唯一索引（非确定性则含 occurrence）。对相同工具调用（如对多个 sub-agent 的多数投票），必须跟踪唯一实例，避免单个 speculated 调用路由到每个副本——除非调用已知确定性。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

## 基准与开销

实现 benchmark 于 OOLONG (trec-coarse, 132k) 和 OOLONG-Pairs (32k)（RLM 原论文），8xH100 80B + vLLM 服务器，Qwen3-30B-A3B-Instruct-0527 在 temperature=0.7 和 0.0，每次 5 遍，4 与 8 并发。**RLM 场景 speed-up 约 1-1.2x**。运行时额外开销可忽略（speculator 廉价地解析检查）；内存上 deepcopy REPL 相对实际变量内存廉价。最坏情况是 serving engine 被大量并发 speculated 请求堵塞。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

## 相关工作与定位

- **Conveyor (Xu et al., 2024)**：允许用户定义部分执行机会（如一行代码），在 decoding 中解析
- **Speculative Interaction Agents (Hooper et al., 2026)**：把 Conveyor 系统正式定义为 "speculative tool calling"，主要减少 TTFT
- **AsyncFC (Feng et al., 2026)**：主张工具调用常被阻塞式实现，定义 future-based async wrapper 契约；风险是与原 harness 轨迹不 1:1

sPTC 认为对更复杂程序中的工具加 speculation 比标准 tool calling 更有用，因为程序运行时未知。标准 tool calling 下，LLM 生成足够 token 完整指定工具调用时，剩余 token 通常不多；而代码执行使实际工具调用模式显著更复杂，留下更多重叠空间。^[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026.md]

实现：https://github.com/alexzhang13/spec-ptc（当前 {Python, bash, Bun} x {Coding harness, RLM, game agent}）。

## 深度分析

### 与 CPU speculative execution 的类比及其边界

三者押注对象不同：CPU 分支预测押控制流，speculative decoding 押 token 分布，sPTC 押数据流中的调用意图——从部分生成的代码提前读出"这个工具迟早会被以这些输入调用"。类比成立处：都是"提前启动 + 命中免等待"，失败代价都是丢弃（speculator 不写真实 REPL 状态）。类比破裂处有三：其一，CPU/speculative decoding 的验证是精确比对，sPTC 的"验证"近乎免费——解析出的调用签名与最终生成一致即缓存可用，真正的风险不在猜错输出，而在猜错输入依赖是否就绪（Case 4 被阻塞的根源）；其二，speculate 边界由人工契约（`speculatable=True, pure=True`、unsafe allowlist）划定，而非全自动硬件机制，更像开发者标注、运行时调度的混合体；其三，CPU speculation 是微秒级，sPTC 处理的是秒级 sub-LLM 延迟，收益空间大一个数量级，这也解释了为何它自称"naive JIT 编译器"。

### 开销结构：为什么 1-1.2x 但仍然值得

OOLONG（trec-coarse 132k / Pairs 32k，Qwen3-30B-A3B，8xH100 + vLLM）给出的 1-1.2x 看似温和，但开销结构是三个"近乎为零"换来的：speculator 只做廉价解析检查（CPU）；shadow REPL 的 deepcopy 相对 132k 上下文变量可忽略（内存）；speculator 不用作真实 executor，错误代码不污染状态、无需回滚（正确性）。因此 speed-up 下限不是被开销吃掉，而是被**命中率**吃掉：可 speculate 的高延迟 sub-LLM 调用占比、streaming 期间输入依赖多早确定，共同决定收益上限。唯一真实风险是 serving engine 被大量并发 speculated 请求堵塞——浪费发生在工具端推理容量而非 harness 端，调优杠杆在 speculate 激进度与工具排队策略。本地部署另有红利：解码 memory-bound 时 speculation 恰好提高 arithmetic intensity，这是云端 batched serving（请求被抽象到独立 engine）拿不到的。

### 何时赢、何时输：任务画像

收益判据可以概括为：**高延迟工具调用占端到端时间比例 × 可提前解析出的调用比例**。

- 赢面大：依赖少量昂贵 sub-LLM/sub-agent 调用的代码执行 harness（RLM 典型）；"think 很久"的长推理模型——主上下文生成越慢，streaming 重叠窗口越大；代码中多个写法串行、数据独立的工具调用（JIT 式并行化）；本地自控 serving 栈。
- 赢面小：工具本身廉价（重叠收益小于复杂度成本）；sub-RLM 等递归成本过高的调用（契约明确排除）；标准 JSON tool calling——调用签名生成完时剩余 token 已不多，窗口天然窄（作者的核心论证：code execution 让调用模式更复杂，反而留出更多重叠空间）；共享 serving engine 且 speculated 请求易堵塞他人的多租户场景。
- 结构性风险：依赖 unsafe 函数（如 `open`）的调用被完全阻塞；非确定性工具需 occurrence 级索引，多数投票场景若不追踪唯一实例，一个 speculated 调用会被错误路由到每个副本。

### 在 latency-hiding 工具箱中的位置

sPTC 与两类常见手法正交且可叠加：prompt caching 是空间换时间，降低每 token 成本，不改变调用间串行结构；并行工具调用（AsyncFC 式 future wrapper）需要模型显式写出并行结构，收益止于模型"知道"的并行度。sPTC 的独特性在于从**部分生成的代码**推断并行性——模型不必写异步代码，harness 替它做 JIT。前继 Conveyor（按行部分执行）与 Speculative Interaction Agents（重 TTFT）证明 decoding 期间部分执行可行，sPTC 把粒度推进到 REPL 命名空间级（Case 3 peekable 依赖），并用 deepcopy shadow REPL 解决"部分执行不污染状态"。与 [[concepts/speculative-decoding]] 同构不同层：后者在 token 层隐藏解码延迟，前者在工具调用层隐藏执行延迟，两者同押"验证便宜、重算昂贵"。它也是 [[concepts/harness-engineering]] 中 latency-hiding 一脉与 [[concepts/inference-optimization]] 的交叉点。

## 实践启示

1. **speculate 契约先行于实现**：先明确哪些工具 `speculatable` 且 `pure`（sub-LLM 可以，sub-RLM 不行），再写解析器。契约错误的代价不是性能损失而是正确性损失（非确定性调用的多个副本被错误路由）。
2. **把整个 REPL cell 当作计算单元**：speculator 与真实 executor 分离、deepcopy shadow REPL 试运行，是"部分执行"类技巧的通用安全模式，可迁移到任何 streaming 解析场景。
3. **盯住 serving 侧拥塞这一唯一真实开销**：基准下限来自命中率而非解析开销；speculated 请求与真实请求共享 engine 时，激进 speculation 会把收益变成他人的延迟。
4. **优先在 code-execution harness 上启用**：标准 JSON tool calling 的重叠窗口天然窄；代码执行让调用模式复杂化，才是 speculation 的主场。
5. **本地部署是收益放大器**：memory-bound 解码下 speculation 提高算术强度；用云端 batched API 则这层收益消失。
6. **与 prompt caching、并行工具调用叠加而非替代**：三者作用在不同层（token 成本 / 显式并行 / 隐式推断并行），组合使用延迟收益近似可加。

## 相关实体

- [[entities/language-model-harnesses-compositional-generalizers-alex-zhang-2026|Language Model Harnesses as Compositional Generalizers (Alex Zhang)]] — 同作者 RLM harness 理论
- [[entities/mit-csail-rlm-harness-length-generalization|MIT CSAIL RLM Harness Length Generalization]] — RLM 上下文
- [[entities/alphaxiv-reinforcement-learning-for-rlms|AlphaXiv RL for RLMs]]
- [[entities/prime-agent-self-improving-rlm-agent|Prime Agent (RLM)]]
- [[concepts/inference-optimization|Inference Optimization]]
- [[concepts/speculative-decoding|Speculative Decoding]] — 更基础的 token 级推测解码
- [[concepts/harness-loop-architecture|Harness Loop Architecture]]

→ [[raw/articles/speculative-programmatic-tool-calling-sptc-alex-zhang-2026|原文存档]]
