---

title: "DeepSeek V4 详解：1M 上下文背后，真正发生了什么"
type: entity
created: 2026-07-04
updated: 2026-09-22
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# DeepSeek V4 详解：1M 上下文背后，真正发生了什么

**来源**: 架构师

**发布日期**: 2026-04-25

**原文链接**: https://mp.weixin.qq.com/s/OkJjRqTwRV2z0Xw9T6Likg

---

## 摘要

DeepSeek V4（preview 版 V4-Pro / V4-Flash，1.6T 总参、49B 激活、1M 上下文）最该细看的不是窗口数字，而是它把"百万上下文够不够便宜"拆成了一串互相咬合的工程问题：注意力 FLOPs、KV cache、共享前缀复用与推理预算被收进同一张成本图。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 目标不是"能跑 1M"，而是让超长上下文变成可常态化支付的生产成本。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

## 核心要点

- **效率账先于能力账**：官方旗语是 "Welcome to the era of cost-effective 1M context length"
- **最硬的数据**：1M 场景下 V4-Pro 相对 V3.2 单 token FLOPs 降到 27%、KV cache 降到 10%，Flash 为 10% 与 7%
- **改的是注意力**：DeepSeekMoE 与 MTP 保留，注意力换 CSA + HCA，残差换 mHC，优化器换 Muon，再叠 FP4/FP8
- **KV cache 被当成存储系统**：classical KV cache 装压缩条目，state cache 装 SWA 与未压缩尾部，还可跨请求复用
- **后训练换路**：先养数学、代码、Agent、指令遵循专家，再用 On-Policy Distillation 合成统一模型
- **边界也写在报告里**：中文白领任务对 Opus-4.6-Max 胜率 53%/10%/37%，复杂指令跟随与多轮写作仍落后 Opus 4.5
- **对做 Agent 的人**：长上下文成本能压、推理预算可分档、搜索与工具被纳进模型级工作流

## 深度分析

长任务 Agent 真正的三笔账是：每轮 prefill 算得起吗（计算账）、KV cache 存得下且能复用吗（系统账）、塞进窗口的东西还在帮忙还是已在干扰（上下文治理账）。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] V4 对前两笔给了数字：1M 场景下单 token FLOPs 降到 V3.2 的 27%（Pro）与 10%（Flash），KV cache 降到 10% 与 7%；以 BF16 GQA8 为基线，V4 系列 1M 下的 KV cache 可压到约 2%。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 报告的野心因此越过"把窗口拉到 1M"：边际成本必须能被压下去，长上下文才可能常态化使用。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

### CSA + HCA：把注意力从原价付费改成分层付费

V4 把注意力拆成两条压缩路线交错使用：**CSA（Compressed Sparse Attention）** 先把每 m 个 token 压成一个 KV entry，再用稀疏选择让每个 query 只看 Top-k 个压缩 KV（压完再挑重点看）；**HCA（Heavily Compressed Attention）** 用更激进的压缩率 m' 但保持稠密注意力。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 交错正是为了同时躲开"太贵"和"丢信息"两个极端。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 基座数据也印证了这条路线：V4-Pro-Base 消化超 32T token，LongBench-V2 40.2→51.5；Flash-Base 只用 13B 激活（约 V3.2-Base 的 35%），多数知识任务已追平甚至反超。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

### mHC、Muon 与 FP4：不是论文装饰，而是配套工程

只看 CSA 与 HCA，容易把 V4 误读成一篇注意力优化论文，PDF 里写得更重。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] mHC（Manifold-Constrained Hyper-Connections）处理深层堆叠的信号稳定性：标准 Hyper-Connections 扩展 residual stream 后容易数值不稳定，mHC 把残差映射矩阵约束到双随机矩阵流形（Birkhoff polytope），谱范数压在 1 以内——CSA/HCA 让长上下文算得起，mHC 让更深更复杂的模型训得稳。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] Muon 在多数模块替换 AdamW，只在 embedding、prediction head 与 RMSNorm 处保留；配套的 fused kernel 与 DualPipe 1F1B 调整把 mHC 的 wall-time overhead 压到 1F1B pipeline stage 的 6.7%。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] FP4 也不是孤立卖点：MoE expert weights 降显存访存压力，CSA indexer 的 QK path 加速超长上下文的 attention score 计算。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

### KV cache：从"分页存张量"到有生命周期的存储系统

PagedAttention 一类方案默认处理规整 KV block，而 V4 的混合注意力会同时冒出多种 KV：CSA/HCA 压缩 KV、Sliding Window 的最近窗口 KV、还没凑够压缩块的 tail state，管理动作从"分页存张量"变成"管理一堆策略不同的状态"。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 报告的 layout 分成 classical KV cache 与 state cache：state cache 给每请求固定大小 block，classical KV cache 按 lcm(m, m') 覆盖原始 token；on-disk KV cache storage 再把压缩 KV、SWA KV、尾部重算策略分开存储，用来消除 shared-prefix 请求之间的重复 prefill。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 这对 Agent 尤其关键——系统提示、工具定义、项目规则、已读文档构成大量共享前缀，前缀越稳定，长任务成本越能被摁在低位；这与 Claude Code 的 Prompt Caching 是同一条线，只是 V4 直接在模型服务侧处理异构 cache。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

### 三件更难的事：报告解了前两件，第三件留给 Harness

Non-think / Think High / Think Max（Max 推荐上下文 ≥ 384K）看着像产品功能，实质是成本控制接口：统一上 Max 既烧钱又拉高延迟，而很多任务上 High 与 Max 差距并不大，Flash 给足 thinking budget 后部分推理能追近 Pro，只是知识类与高难 Agent workflow 仍会落后。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] tool-calling 上 V4 跨多轮保留完整 reasoning content（V3.2 每个新用户 turn 就丢弃），普通对话仍会丢弃旧 reasoning——工具链路需要累积任务线索，聊天一直保留只会把历史想法越堆越重。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] 搜索侧同理：Non-think 走传统 RAG，Thinking 走 agentic search，后者对 RAG 胜率 61.7% vs 18.3%（平 20.0%），平均 16.2 次工具调用，成本只是边际增加——搜索不再是外挂的检索模块，而是推理过程的一部分。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

### 与竞品的对比，以及那句"调侃 Claude"

报告拉 Claude Opus 4.6 Max 对比白领任务，并留下长文生成"不像对手那样糊 bullet points"的评语；更值钱的是它自建的中文高级职业任务：30 个任务覆盖 13 个行业，跑在带 Bash 与 web search 的内部 harness 上人工评分，总体胜率 53.0% / 平 10.0% / 负 37.0%，内容质量领先而指令遵循略低，对 Opus 4.5 在多轮写作与复杂指令跟随上是 45.9% vs 52.0% 落后。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

中文功能写作对 Gemini-3.1-Pro 为 62.7% vs 34.1%，原因是后者有时让固有风格偏好覆盖用户明确要求——企业场景的写作能力其实是"听懂约束、保住结构、符合本地格式"的组合。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md] V4 双线开源并用 FP4 QAT 把 Instruct 仓库尺寸近乎腰斩，但报告也自曝边界：85 位内部开发者中 52% 认为 V4-Pro 可当默认主力 coding model、39% 倾向是、不到 9% 说否，问题清单是误读模糊 prompt 与 occasional over-thinking——模型越强，外层 Harness 的责任越重。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

## 实践启示

1. **模型、模式、上下文长度与工具预算一起纳入路由**：低风险任务走 Flash + Non-think，规划与代码审查走 High，数学与关键设计评审才升 Max。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]
2. **先算自己的三笔账**：每段上下文是否每轮重算？有没有稳定前缀？尾部能否压缩？这比"该不该换模型"更接近真实成本。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]
3. **按共享前缀组织上下文**：system prompt、工具定义、项目规则、已读文档保持稳定，动态内容往尾部生长。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]
4. **reasoning 的留与丢按场景分**：工具链路保留完整推理历史以维持累积线索，普通对话及时丢弃旧 thinking。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]
5. **接入前补齐 Harness**：沙箱、测试闭环、超时重试、来源质量验证、可回滚与审计。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]
6. **评测按任务切而非按品牌切**：中文成稿型任务 V4 更主动完整，复杂约束遵循与多轮稳定性 Claude 仍硬；Base 用于微调研究、Instruct 用于产品接入。^[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么.md]

→ [[raw/articles/deepseek-v4-详解1m-上下文背后真正发生了什么|原文存档]]

## 关联

- 相关实体: [[entities/deepseek-v4|DeepSeek V4]]、[[entities/deepseek-v4-pro-vs-claude|DeepSeek V4 Pro vs Claude]]
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]、[[concepts/context-window-economics|Context Window Economics]]
- 工程延伸: [[entities/anthropic-prompt-caching-claude-code|Claude Code Prompt Caching]]、[[entities/deepseek-cost-migration-system-layer-kv-cache-harness|KV Cache 与 Harness 成本迁移]]
