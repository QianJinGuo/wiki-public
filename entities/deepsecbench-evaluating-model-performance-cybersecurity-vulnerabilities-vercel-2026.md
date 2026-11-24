---
title: "DeepSecBench：评估 AI 模型在网络安全漏洞发现中的性能"
created: 2026-07-29
updated: 2026-11-24
type: entity
tags: [ai, security, benchmark, vulnerability, deepsec, vercel, llm-evaluation]
sources: [raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026]
confidence: 0.72
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# DeepSecBench：评估 AI 模型在网络安全漏洞发现中的性能

> **Background**：本文基于 Vercel 发布的 DeepSecBench 基准测试。该基准评估了多个 AI 模型（Claude、GPT、Gemini 等）在自动化网络安全漏洞发现上的表现，并探讨了模型能力与安全扫描自动化之间的关系。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md]

## 核心洞察

DeepSecBench 是一个评估 AI 模型在代码安全漏洞发现中性能的标准化基准。它通过构建受控的漏洞环境，测试模型识别、分析和利用安全漏洞的能力。与传统的安全扫描不同，DeepSecBench 关注的是模型在端到端漏洞发现流程中的表现，而非单纯的特征匹配。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md]

## 关键发现

- **模型能力差异显著**：不同模型在漏洞发现任务上的表现差异很大，顶尖模型（如 Claude Opus 4、GPT-5.5）在特定漏洞类型上表现出色，但在其他类型上仍有明显短板。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md]
- **成本与性能的权衡**：更强的模型通常意味着更高延迟和成本。DeepSecBench 提供了成本效益分析框架，帮助团队在安全扫描场景下做出合理的模型选型决策。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md]
- **上下文长度的影响**：安全漏洞发现需要在完整的代码上下文理解才能准确识别问题。模型的理解深度与上下文窗口大小正相关。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md]

## 方法论

DeepSecBench 使用受控的沙箱环境，在其中植入已知漏洞，然后评估各个模型自主发现、分析和利用这些漏洞的能力。测试过程降低了 guardrails 以模拟真实的安全研究场景。这种方法提供了标准化的对比数据，避免了传统安全基准测试中环境差异带来的评估偏差。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md]

## 深度分析

### 基准测什么：召回加权的 F2 分数与防污染设计

DeepSecBench 的评估对象不是"模型能否写安全报告"，而是端到端的漏洞发现能力：在开源代码库被大规模修复漏洞前的某个 commit 上，选定 50 个入口文件，以 231 条人工判定的发现构成 golden set，考核模型能否自主把它们找出来。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:29] 计分公式为 recall 加权的 F2（Score = 100×5PR/(4P+R)），召回权重是精确率的两倍——漏报的漏洞不会被修复，而误报只是浪费人力，不降低代码库安全性。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:29] 这一权重选择本身就是一种价值判断：安全扫描场景下"宁多报、勿漏报"。对 golden set 之外的发现，由裁判模型判定真假并计入精确率；仓库、commit、文件与发现全部保密，杜绝模型背题——即便如此，最好的单次运行也只有 30.7% 召回，25 次运行中 20 次低于 20%，说明该任务远未饱和。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:31-35]

### 排行榜解读：顶级分数靠堆推理，性价比在中档模型

排行榜（25 个模型×推理档位组合）呈现明显的分层。榜首 GPT-5.6 Sol（xhigh）得分 35.58，成本 $55.98、耗时 3 小时 39 分；Claude Opus 5（max）以 32.57 次之，但成本飙到 $127.93；Kimi K3（high）排第 8，得 17.56 分只花 $12.38；Grok 4.5（high）15.58 分仅 $5.60。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:47-58] 一个反直觉的发现是"更贵不等于按比例更强"：Kimi K3 用约五分之一的价格拿到一半的分数，Grok 4.5 用不到一半的价格拿到接近 Kimi 的表现。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:41] 同一模型的推理档位也在重塑性价比曲线：GPT-5.6 Sol 从 xhigh 降到 medium，分数从 35.58 降至 25.10，但成本近乎砍半（$17.95）、时间从近 4 小时缩到 31 分钟，且精确率反而从 96.3% 升至 94.3%（另一组可比数据），成为顶级模型中分数/成本比最优的组合。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:53] 值得注意的是精确率与召回率的分化：GPT-5.5（xhigh）召回仅 17.7% 但精确率 95.5%，Kimi K3 精确率只有 77.1% 却召回 14.7%——两者组合风格截然不同。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:54-56]

### 方法论的边界：单仓库、单语言、外部有效性存疑

方法论有三处值得警惕的局限。其一，基准跑在**一个**秘密仓库上，50 个入口文件、231 个发现全部来自同一代码库，结论对其他语言、框架、漏洞类型的可迁移性（外部效度）未经证明；单仓库的高分可能反映的是对该仓库代码风格与漏洞分布的适应，而非通用漏洞发现能力。其二，golden set 由"人工判定"构成，gold 标准本身的噪声会直接进入分数；golden set 之外的发现交给裁判模型判真假，这意味着精确率部分依赖另一个模型的判断，存在系统性偏差被放大的风险。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:31] 其三，报告成本是 50 文件的一次扫描开销；真实生产代码库动辄是它的 100 倍规模，一次全量扫描 Kimi K3 约 $1,200、顶级 OpenAI 模型超 $5,000，且这只是单次——安全扫描是持续性的重复工作，实际预算要乘上扫描频率。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:75] 此外，Anthropic 最强模型 Fable 5 因拒绝安全类任务（含防御性任务）缺席榜单，说明"谁在榜上"本身受模型厂商政策过滤，基准并不代表全模型生态。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:45]

### 对 AI 辅助安全工作的含义：防守方信息优势 + 分层扫描

基准的底层叙事是一个不对称博弈：攻击者再强也只能从外部探测，防守方握有源码、架构与历史的完整可见性——同一个模型在知情者手里比在攻击者手里更强，"模型能读源码，找到的是攻击者只能试探的东西"。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:95] 这个优势成立的前提是防守方真的先用了模型。把排行榜放进工程语境，可行做法是分层的扫描程序：frontier 模型做低频深审计，Kimi K3 / Grok 4.5 这类中档模型跑高频常规扫描；GPT-5.6 Sol 降到 medium 后 31 分钟一轮，已可用于新功能合并前的快速审查。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:79] 文章给出的两档参考架构：初创公司每次合并用 Grok 4.5、里程碑版本才上贵模型；大企业则在关键服务跑 frontier 审计、关键 PR 用同款模型低档推理、其余代码库交给 Kimi/Grok 级别的持续扫描。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:81] 基准自身则通过 AI Gateway 单端点路由全部模型，自动处理路由、重试与故障转移——突发高 token 消耗的安全扫描负载，靠单一供应商直连很难买到足够的突发容量。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:85]

## 实践启示

1. **扫描选型先看分数/成本比，不看榜单名次。** Kimi K3（第 8 名）与 Grok 4.5（第 10 名）以 1/5 到 1/10 的成本提供接近一半的分数，是持续扫描预算下的默认候选；frontier 榜首留给低频深度审计。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:41]
2. **用推理档位调性能-延迟曲线，而不是换模型。** GPT-5.6 Sol xhigh→medium：35.58/3h39m → 25.10/31min，同一模型即可在"深度审计"与"合并前快扫"两种任务间切换，选型时把档位当作独立变量列出。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:79]
3. **把召回敏感的 F2 权重搬进自己的验收标准。** 安全扫描的评估指标应复制"漏报惩罚加倍"的价值取向：工具漏掉的漏洞不会自己修复，误报只是人工复核成本；评估自建扫描流水线时同样以召回为先。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:29]
4. **别照抄基准绝对分数，先校准外部效度。** 结果来自单一秘密仓库、F2 计分、三次取中位数与裁判模型判分，换成你的代码库后分数会漂移；先在自己仓库上抽样复测，再决定预算分配。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:33]
5. **预算按"文件数×扫描频率"估算，而不是按单次基准成本。** 基准成本只覆盖 50 个文件；十万行级代码库单轮全量扫描 $1,200（Kimi K3）到 $5,000+（frontier），持续程序需要乘上扫描频率，按月预算而不是按次。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:75]
6. **防守优势只属于先动手的一方。** 模型同等可用时，先找到漏洞的一方决定攻防结果；把 AI 扫描接进 CI/合并流程的时机本身就是安全投资，拖到攻击者先用的那一天，信息优势归零。^[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026.md:95]

## 相关实体

- [[entities/introducing-deepsec-find-and-fix-vulnerabilities-in-your-code-base|Introducing deepsec: The security harness for finding vulnerabilities in your codebase]]
- [[entities/cybersecqwen-4b|CyberSecQwen-4B]]
- [[entities/拆解openclaw架构七安全漏洞阿喀琉斯之踵|拆解 OpenClaw 架构（七）：安全漏洞]]
- [[entities/agentic-penetration-testing-legal-questions|Agentic Penetration Testing Legal Questions]]
- [[entities/multimodal-evaluators-mllm-as-judge-image-to-text|Multimodal Evaluators：MLLM-as-Judge]]

→ [[raw/articles/deepsecbench-evaluating-model-performance-cybersecurity-vulnerabilities-vercel-2026|原文存档]]
