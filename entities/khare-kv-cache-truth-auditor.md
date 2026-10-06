---
title: "A cache hit is not proof that you skipped the work"
created: 2026-09-22
updated: 2026-10-06
type: entity
tags: [kv-cache, inference, correctness, auditing]
sources: [raw/articles/khare-kv-cache-truth-auditor]
confidence: 0.7
---

# A cache hit is not proof that you skipped the work

## 摘要

KV-cache 正确性审计工具：cache hit 不等于 prefix 复用——确定性测试暴露 exact-dup/interior-mutation/namespace-isolation 三种 reuse 语义，cached 与 no-cache 输出 token-identical 验证^[raw/articles/khare-kv-cache-truth-auditor.md]

## 核心要点

- v*c=49（value=7, confidence=7, stars=4），newsletter ingest 2026-09-22
- 详细分析见原文存档

## 深度分析

### 三种 reuse 语义把"缓存命中"拆成三个不同的问题

文章最有价值的动作是把笼统的 cache hit 拆成三种可分别验证的 reuse 语义：exact-dup（逐 token 完全一致才复用）、interior-mutation（prompt 中段一旦变化，safe prefix 在该处截断，后续全部重算）、namespace-isolation（隔离命名空间之间互不共享，复用为零）。三类在 ten-case 控制实验中分别对应 verified_hit、partial_reuse、verified_miss 三种 typed verdict——verdict 不是 hit/miss 二值，而是带证据等级的连续谱^[raw/articles/khare-kv-cache-truth-auditor.md]。这直接反驳了常见工程直觉："prompt 看起来差不多"不等于"可以复用"；同样，"miss" 也可能是 eviction、namespace 隔离或真正 cold 三种不同原因，需要区分报告而不是合并成一个 miss 指标^[raw/articles/khare-kv-cache-truth-auditor.md]。

### 量化边界：8/9、3/9、0

用九 token 的固定请求序列，文章给出清晰量化边界：exact-duplicate 复用 8 个 token、只做 1 个 token 的 prompt work；interior mutation 把复用截断到 3、重算 6；suffix 变化只影响最后一个 token，仍保留 8 个安全 token；same-length-different-ids 是最隐蔽的陷阱——长度相同但 token ID 不同，复用为零、全部重算^[raw/articles/khare-kv-cache-truth-auditor.md]。复用边界由 exact token identity 决定，而非相似度或长度；prompt 中段任何改动（哪怕语义等价）都会把复用边界推回改动点。这与 [[entities/anthropic_cache_tokenomics|cache tokenomics]] 中"易变内容放尾部"的实践在机制上一致^[raw/articles/khare-kv-cache-truth-auditor.md]。

### 独立 oracle 而非 engine 自证：五项检查的 fail-closed 链

方法核心洞见：engine 不能自己验证自己。五项独立检查构成 fail-closed 证据链——engine attestation（被测陈述）、token oracle（从 exact token identity + cache policy 推导的期望复用前缀）、observed prompt work（实际处理的 token 数）、output identity（cached 与 no-cache 输出 token-identical）、evaluator（确定性任务结果正确），最后由 hash-bound verifier 校验 schema/artifact/checksum 一致性；五者全部吻合，attestation 才升级为 evidence^[raw/articles/khare-kv-cache-truth-auditor.md]。这是把 [[concepts/verifier-driven-development|verifier-driven development]] 的"声明与验证分离"原则应用到 KV-cache：reuse 计数正确 ≠ 输出没变，输出 token-identical ≠ 任务正确——三个问题正交，各自需要独立检查^[raw/articles/khare-kv-cache-truth-auditor.md]。

### evicted 是 typed verdict，不是笼统 miss

capacity-revisit case 展示了证据链的精细度：同一 4-token 请求 revisit 时复用为零，但 verdict 不是 verified_miss 而是 `evicted`——依据是保留的 predecessor proof 和受控 residency 观察，证明早先的候选条目确实离开过缓存^[raw/articles/khare-kv-cache-truth-auditor.md]。运维价值在于：eviction 误报成 miss 会掩盖容量配置问题，namespace 隔离误报成 miss 会掩盖路由/隔离配置问题。typed verdict 让"为什么没命中"成为可审计的答案。文章同时诚实标注证据边界：这是 token 级确定性合成控制，只验证 auditor/oracle/attestation/evaluator/verifier 本身，不证明 MLX 或 vLLM 的实际 speedup、生产缓存正确性或 GPU 性能——"null is not zero"，未测量不等于为零^[raw/articles/khare-kv-cache-truth-auditor.md]。

## 实践启示

1. **不要把 cache hit 当作"跳过了工作"的证明。** 复用计数、实际跳过的计算、输出一致性是三个正交命题；上线依赖 cache 的功能前，至少核对 observed prompt work 与 attestation 是否吻合^[raw/articles/khare-kv-cache-truth-auditor.md]。
2. **写 prompt 时把易变内容放尾部。** 中段任何变化都会把复用边界截断到改动点（8/9 → 3/9），suffix 变化只损失最后一个 token；时间戳、随机 ID、用户名等动态内容应集中在 prompt 末尾。
3. **警惕 same-length-different-ids 陷阱。** token 数相同不代表 prefix 相同；缓存命中率归因分析要按 exact token identity 对齐请求，而不是长度或字符串相似度。
4. **把 miss 拆成 typed verdict 再归因。** cold miss、eviction、namespace isolation 应分开统计，否则容量规划和隔离配置问题会被平均化进一个 miss 率指标^[raw/articles/khare-kv-cache-truth-auditor.md]。
5. **为缓存正确性建立独立 oracle。** runtime 自报命中率只是信号不是证据；用确定性合成控制（固定 token 序列 + 已知期望复用）对缓存层做回归测试，成本低、信号强^[raw/articles/khare-kv-cache-truth-auditor.md]。
6. **区分"未测量"与"测量为零"。** 引用性能数字前先检查证据边界：该工具验证的是审计链本身，MLX/vLLM 真实 speedup 属于未测量范围，不可据此估算生产收益。

相关：[[raw/articles/mooncake-cluster-level-kv-cache-pooling-production-2026-09-11]]

→ [[raw/articles/khare-kv-cache-truth-auditor|原文存档]]
