---
title: "OpenRouter 2026 开源 F4：DeepSeek V4 Flash、GLM 5.2、MiniMax M3、Nemotron 3 Ultra 全景分析"
type: entity
tags: [openrouter, open-source, model-comparison, deepseek, glm, minimax, nemotron, nvidia, llm, inference]
created: 2026-07-01
updated: 2026-11-13
sources: [raw/articles/openrouter-f4-open-source-models-analysis-2026]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OpenRouter 2026 开源 F4：DeepSeek V4 Flash、GLM 5.2、MiniMax M3、Nemotron 3 Ultra 全景分析

> 来源：[[raw/articles/openrouter-f4-open-source-models-analysis-2026|原文存档]]

OpenRouter 于 2026 年 6 月发布了《The Open-Weight Models That Matter》报告，整理出截至 2026 年 6 月最值得关注的 4 个开源模型（"开源 F4"）——DeepSeek V4 Flash、GLM 5.2、MiniMax M3、NVIDIA Nemotron 3 Ultra。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

## DeepSeek V4 Flash：性价比之王

DeepSeek V4 Flash 是第一个被开发团队直接塞进智能体工作流的开源模型。2820 亿总参数 / 130 亿激活的 MoE 架构，SWE-bench Verified 得分 79.0%（Pro 版 80.6%）。采用 MIT 协议，支持百万 token 上下文。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

- **智力指数**：40（Artificial Analysis）
- **价格**：输入 $0.054/M token，输出 $0.242/M token（含缓存后低至 $0.029/M 输入）
- **速度**：约 84 tokens/秒
- **适用场景**：极致性价比的智能体与代码生成
- **避坑**：写文章与语气把控一般，提示词需非常具体^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

## GLM 5.2：开源质量铁王座

GLM 5.2 于 2026 年 6 月中旬发布，在 Artificial Analysis 4.1 智力指数拿下 51 分排名开源第一，距离闭源 Claude Fable 5 仅差 5 分。MIT 协议，真实智能体基准测试中与 GPT-5.5 xhigh 打平。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

- **智力指数**：51（开源第一）
- **价格**：输入 $0.447/M token，输出 $3.31/M token
- **速度**：约 78 tokens/秒
- **核心优势**：复杂任务规划与超长上下文代码编写
- **避坑**：思考过程消耗大量输出 token，刚发布不久各家托管平台质量参差不齐^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

## MiniMax M3：多模态长文本专精

MiniMax M3 是 F4 中唯一原生理解文本、图表和视频的模型。智力指数 44（与 DeepSeek V4 Pro 并列），但在真实智能体测试中与 Claude Sonnet 4.6 持平。社区协议（非 MIT），商业使用需署名。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

- **智力指数**：44
- **价格**：输入 $0.098/M token，输出 $1.21/M token（超 51 万 token 上浮）
- **核心优势**：多模态理解——屏幕截图、UI 界面、架构图、视频
- **适用场景**：UI 自动化测试、看图写代码、图文文档解析、视频工作流
- **避坑**：话痨型模型，推理过程很长；社区协议非 MIT^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

## NVIDIA Nemotron 3 Ultra：企业级开源

Nemotron 3 Ultra 是美国本土最能打的开源模型。5500 亿总参数 / 550 亿激活的 Mamba-2 + Transformer 混合 MoE，采用 NVFP4 精度，支持百万 token 上下文和多 token 预测，OpenMDW 协议。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

- **智力指数**：48（开源第二）
- **价格**：输入 $0.423/M token，输出 $2.61/M token（有免费测试通道）
- **核心优势**：NVIDIA 软硬件生态支撑，极高部署效率
- **避坑**：纯文本模型，基础智力不如 GLM 5.2，极限代码略逊中国开源头部^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

## 总体评估

开源与闭源的差距在过去 18 个月中稳定保持在 3 到 6 个月之间。闭源大厂完全没有甩开开源阵营的迹象。随着企业 AI 用量激增，控制成本成为核心诉求，开源模型迎来了真正的高光时刻。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md]

→ [[raw/articles/openrouter-f4-open-source-models-analysis-2026|原文存档]]

---
## 深度分析

### 价格带与真实成本：单价低不等于总价低

F4 报价表只是成本测算的起点。DeepSeek V4 Flash 在官方 API 上输入 $0.14、输出 $0.28，叠加缓存折扣后输入低至 $0.029，且自 2026 年 5 月起固化为永久价格，输出成本约为 GPT-5.5 的 1/150；走西方第三方托管约贵一倍，但相对其智力水平仍是白菜价。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md:49] 反向案例是 GLM 5.2：输出时喜欢深入思考、消耗大量 token，单价不高但真实总价随任务深度放大。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md:63] MiniMax M3 则是话痨型模型、推理过程很长，且上下文超过 51 万 token 后价格还会上浮。成本评估应按"单价 × 输出 token 放大系数 × 阶梯阈值"建模，这是 [[concepts/ai-cost-optimization-framework|AI 成本优化]] 的基本功。

### 四象限生态位：按业务约束对号入座

F4 并非同质竞争，而是四个几乎不重叠的生态位：Flash 以极致性价比跑出闭源头部级代码能力，GLM 5.2 主攻复杂规划与代码库级重构质量，M3 独占原生多模态长文本，Nemotron 3 Ultra 面向企业级私有化部署。原文的选型法则是先弄清业务到底需要极限低价、顶级质量、多模态能力还是企业级部署，再对号入座测试，而非追逐单一榜单。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md:99]

### 版本梯度与协议光谱

DeepSeek 用 Flash/Pro/1.6 万亿参数的超大杯版本覆盖同一能力族，相邻档位 SWE-bench 分差仅约 1.6 分——先以 Flash 起步、确需极限性能再上 Pro，是理性的默认路径。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md:47] 协议端则拉开光谱：MIT（DeepSeek、GLM 5.2）最开放，OpenMDW（Nemotron）次之，M3 的社区协议最收紧，商用需署名、大型商业产品需书面授权。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md:79] 协议宽松度应与托管质量、上下文支持一起纳入供应商尽职调查，模型细节另见 [[entities/minimax-m3-frontier-open-source-model|MiniMax M3 专题]]。

### 地缘政策与厂商生态的双重推手

需求侧，出口管制新规迫使 Anthropic 大规模禁用 Fable 5 与 Mythos 5 的海外访问权限，MIT 协议、代码能力逼近闭源头部的 GLM 5.2 随即成为追求业务稳定性企业的替代选项——地缘政策正在直接为开源模型导入企业级需求。^[raw/articles/openrouter-f4-open-source-models-analysis-2026.md:65] 供给侧，NVIDIA 将训练数据、配方、评估工具与 RL 基础设施随模型全量开源，本质是以开源模型为自家芯片与软件全家桶获客，且有硬件营收支撑长期投入。叠加企业 AI 用量激增的降本诉求，开源阵营同时获得需求、供给与成本三重顺风。闭源对照能力可参考 [[entities/ai-agent-rli-benchmark-fable5-gpt55-comparison-2026|GPT-5.5/Fable 5 智能体基准]]。

### 速度与托管质量：部署侧的隐藏变量

四款模型生成速度从约 59 到 84 tokens/秒不等：Flash 最快，GLM 5.2 约 78，Nemotron 约 75，M3 最慢且推理冗长，长任务时延差距被进一步放大。托管侧的变量同样实际存在：GLM 5.2 刚发布时各家平台质量参差不齐，M3 的全量上下文支持因服务商而异，Nemotron 免费通道只能测试、撑不起正式商业产品。同一模型在不同供应商处的实测表现，可能比榜单分差更影响落地效果。

## 实践启示

1. **先定约束再选模型**：把业务需求归入极限低价、顶级质量、多模态、企业级部署四象限，对号入座做小规模测试，而不是按榜单排名跟风迁移。
2. **按 token 结构而非单价测算成本**：思考型模型（GLM 5.2、M3）的输出放大效应会让"单价便宜"变成"总价更贵"，用真实 agent 负载回归并叠加缓存价与阶梯阈值，测算稳态成本。
3. **默认从 Flash 档起步**：同族版本分差仅约 1.6 分，先以最低价版本跑通工作流，确认性能瓶颈确实存在后再升级大杯，避免为边际性能支付数倍价格。
4. **把协议当风险清单审查**：MIT 可放心商用；社区协议与 OpenMDW 项目先核对署名要求、书面授权条款与商用边界，再决定是否进入生产环节。
5. **把地缘合规纳入供应商对冲**：闭源模型的海外访问可能被政策性禁用，MIT 协议模型加上多供应商托管是业务连续性上成本最低的保险。
6. **托管平台质量是隐藏变量**：同一模型在不同托管方表现参差（GLM 5.2 刚发布时尤甚），切换供应商前先在小样本任务上回归评测，再全量迁移。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

