---
title: "都在卷「让大模型多循环几遍」，这个7B模型LoopCoder v2说：多循环 1 次就够了"
created: 2026-07-01
updated: 2026-11-10
type: entity
tags: [ai, llm]
sources: [raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了]
confidence: 0.9
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 都在卷「让大模型多循环几遍」，这个7B模型LoopCoder v2说：多循环 1 次就够了

--- ^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]
wechat_mp_fakeid: MP_WXS_3073282833^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


## 都在卷「让大模型多循环几遍」，这个7B模型LoopCoder v2说：多循环 1 次就够了

当所有人都在比谁「想得更久、算得更多」——推理模型动辄输出成千上万个思考 token，循环式架构恨不得在内部反复迭代十遍八遍——一项新研究反手泼了盆冷水：  一个 7B 的小模型，只需要在正常计算之外「多循环这一次」（总共 2 次），就能在号称最难的真实代码修复基准 SWE-bench Verified 上从 43.0 分飙到 64.4 分；而继续往上加循环，不仅不涨，反而一路跳水。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


论文标题起得很干脆——《Only Loop Once》，只循环一次。背后是来自  北京航空航天大学、IQuest Research、澜舟 ^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

## 核心要点

> 本文为微信公众号文章，由 WeChat backfill 收录。

## 详细信息

---
wechat_mp_fakeid: MP_WXS_3073282833^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

--- ^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

## 都在卷「让大模型多循环几遍」，这个7B模型LoopCoder v2说：多循环 1 次就够了

当所有人都在比谁「想得更久、算得更多」——推理模型动辄输出成千上万个思考 token，循环式架构恨不得在内部反复迭代十遍八遍——一项新研究反手泼了盆冷水：  一个 7B 的小模型，只需要在正常计算之外「多循环这一次」（总共 2 次），就能在号称最难的真实代码修复基准 SWE-bench Verified 上从 43.0 分飙到 64.4 分；而继续往上加循环，不仅不涨，反而一路跳水。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


论文标题起得很干脆——《Only Loop Once》，只循环一次。背后是来自  北京航空航天大学、IQuest Research、澜舟科技和中国人民大学  的联合团队。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


* ** 论文标题：  ** LoopCoder-v2：Only Loop Once for Efficient Test-Time Computation Scaling

* 论文地址：  https://arxiv.org/pdf/2606.18023

* 研究团队：北京航空航天大学 · IQuest Research · 澜舟科技 · 中国人民大学

* ** 模型主页（HuggingFace）：  ** ` huggingface.co/Multilingual-Multimodal-NLP/LoopCoder-V2  `

▲ 核心结论一图流：多循环带来「精修收益」，也带来几乎恒定的「位置错配成本」；收益在第 2 次循环达到峰值后迅速衰减，于是「只循环一次（共 2 次）」成为最优解。 ^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

###  一、「循环」，当下最热的卷法


自从 o1、Claude 这一代推理模型把「想得越久越强」写进行业共识，  「测试时计算」（test-time compute）就成了过去一年最热的方向：  与其把模型练得更大，不如让它在推理时多花点算力，把答案反复打磨。要理解这项研究，先得知道大家具体在卷什么。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


过去想让模型更强，常规操作是把网络堆得更深、参数更多。而「循环式」大模型（Looped / Recurrent-depth LLM）换了个思路：不堆新层，而是让  同一套参数，在「脑子里」把内部表征反复打磨好几遍。  打个比方，这就像同一个人把一道题在心里默默重算几遍，而不是请来更多人、或者把草稿纸写满——它是一种省参数的「测试时计算」（test-time compute）。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


听起来很美，但有个硬伤：  顺序循环太贵。  每多循环一次，就要多走一遍计算，延迟和 KV-cache 显存都跟着循环次数线性上涨。想多循环，算力扛不住。 ^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

并行循环 Transformer（Parallel Loop Transformer，PLT）就是为了解决这个问题。它用两招把成本摁了下去：一是 CLP（跨循环位置偏移），打断循环之间的串行依赖，让多次循环可以并行计算；二是 G-SWA（共享 KV 的门控滑窗注意力），让显存几乎不随循环次数增长。成本被压平之后，  「循环几次」第一次变成了一个可以自由拧的旋钮。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


###  二、旋钮拧大 ≠ 更强：

###  第 2 遍封顶，第 3 遍跳水


问题来了：这个旋钮，到底拧到几合适？

团队干脆从零训了一整个家族：7B 稠密模型，  18T token、文本与代码 1:1、覆盖 100 多种编程语言，前后烧掉约 100 万 GPU 小时。  唯一的变量，就是循环次数。结果非常反直觉：^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


** 模型（均为 7B）  ** |  ** SWE-bench Verified  ** |  ** SWE-bench 多语言  ** |  ** LiveCode Bench  ** |  ** 平均分  **^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

---|---|---|---|---
不循环 Baseline（1 次）  |  43.0  |  14.0  |  27.4  |  38.0^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

** LoopCoder-v2（2 次）★  ** |  ** 64.4  ** |  ** 31.0  ** |  ** 35.4  ** |  ** 46.5  **^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

LoopCoder-v2（3 次）  |  27.6  |  11.0  |  28.6  |  36.9^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]

LoopCoder-v2（4 次）  | ^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md]


## 深度分析

### 收益侧：第 2 遍循环包办了全部「精修」

团队用三把探针——隐状态演化、注意力路由、输出分布变化——同时观测每次循环，只有三者一致才下结论。第 2 次循环几乎包办所有有用的精修：隐状态同向稳步收敛、注意力重新分配、输出分布改善，表征多样性（effective rank）也在此步达到峰值；此后更新越来越小甚至方向反转，注意力路由基本「冻结」，循环近乎空转。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md:69-71]

### 成本侧：CLP 引入几乎恒定的「位置错配税」

CLP 为让循环可并行做了取巧：从第 2 次循环起，每个 token 拿到的是「邻居」上一轮的状态而非自己的，由此引入固定的位置错配成本。团队把它量化为可从模型内部状态直接算出的指标 Ω，发现它在每次循环几乎恒定、不随循环次数摊薄。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md:73-75]

### 「只循环一次」是收益递减与成本恒定的交点

精修收益在第 2 次循环后崩塌且未再恢复，错配成本高位横盘；超过 2 次后性能掉头向下——SWE-bench Verified 从 64.4 跳水到 27.6（3 次）、22.4（4 次），平均分甚至不如不循环。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md:56-59,80] 「循环几次最好」不必暴力扫 benchmark，内部诊断指标即可提前看出——与 [[entities/openai-astra-looped-transformer-technical-clarification-raschka-2026|looped transformer 技术澄清]] 一脉相承：关键变量是循环策略而非次数。

### 越级对比：7B 逼近大它几十倍的旗舰

这个 7B 模型在 SWE-bench Verified 拿到 64.4 分，超过 2350 亿参数的 Qwen3-235B（45.2），逼近 Kimi-K2（69.2）与 Qwen3-Coder-480B（67.0）；agentic 任务提升尤其明显：Terminal-Bench 两版本 26.3→34.2、11.2→21.0，BFCL 32.2→40.1——跃升靠的不是更大的模型，而是「多想了一遍」。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md:61-63]

## 实践启示

1. 给「测试时计算 = 越多越好」踩刹车：PLT 这类并行循环架构存在明确且偏低的甜点（共 2 次），盲目加循环是负收益，与 [[entities/gka-variable-test-time-compute-2026|按任务分配测试时计算]] 互为印证。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md:82-88]
2. 选超参先看内部指标：用可解释性诊断（收益/成本曲线、Ω）替代暴力扫 benchmark，可迁移到推理预算、迭代轮数等其他测试时计算手段的分配上。
3. 小模型性价比路线：7B 多循环一次即可在真实软件工程任务上越级，对端侧和算力受限场景比堆参数划算；同类思路还见于 [[entities/yifanzhang-rlt1-recurrent-looped-transformer|RLT1 循环 Transformer]]。
4. 并行化的代价要显式记账：CLP 用「邻居状态」换并行性，换来恒定错配成本；为并行取巧的改动都应量化自己的「税」。
5. 迭代收益要测边际而非假设：收益集中在第 2 遍（effective rank 峰值），后续近乎空转——任何「反复打磨」流程都应实测边际收益并设止损点（结合 [[concepts/inference-optimization|推理优化]] 算账）。
6. 后续方向：自适应位置偏移、按任务动态分配循环次数、内部循环与显式 CoT 的配合，都是把「循环」从固定旋钮变成调度策略的切入点。^[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了.md:90]

## 原文

→ [[raw/articles/都在卷让大模型多循环几遍这个7b模型loopcoder-v2说多循环-1-次就够了|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

