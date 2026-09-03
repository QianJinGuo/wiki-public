---
title: LLM 可观测性 4 层模型与工具栈
created: 2026-06-10
updated: 2026-08-01
type: concept
tags: [llm, observability, otel, langfuse, langsmith, dcgm, eval, monitoring, genai-semantic-conventions]
confidence: 0.9
provenance_state: merged
stars: 5
value: 9
sources: [raw/articles/llm-observability-4-layer-quant67]
---

# LLM 可观测性 4 层模型与工具栈

LLM 系统的可观测性不是把传统微服务的 Metrics + Logs + Traces 搬过来就行 — 计费单位是 token×单价而不是 CPU 秒；延迟要拆 TTFT/TPOT/E2E；HTTP 200 不代表答案对。**4 层观测模型** 由 quant67 2026 系列文章提出，是当前最系统化的方法论。

→ [[raw/articles/llm-observability-4-layer-quant67|原文存档]]

## 核心论点

可观测性的目标不是"看到指标"，而是**"出问题时 5 分钟定位、3 小时修复，下次不再出现"**。

围绕这个目标，既要讲指标口径和工具选型，也要给出告警阈值、故事复盘、平台化落地路径。

## 4 层模型

```
[业务层]   Cost / A/B / Compliance / 留存
[质量层]   Eval / 幻觉 / 用户反馈 👍/👎
[调用层]   Trace: LLM/Tool/Retriever / Prompt 版本 / TTFT/TPOT/E2E
[基础设施] GPU DCGM / vLLM SGLang metrics / KV cache / Prefix hit
```

**自下而上**展开。Llm Observability 4 Layer Quant67#^Block 2

## 与传统微服务的 8 大差异

| 维度 | 传统微服务 | LLM 服务 |
|---|---|---|
| 计费单位 | 请求数 / CPU 秒 | input/output/cached token，按模型不同价 |
| 延迟语义 | 单一 latency（p50/p99） | TTFT（流式首包）+ TPOT（每 token）+ E2E |
| 正确性 | 状态码 200 即可 | 200 不代表正确，可能幻觉/跑题/拒答 |
| 调用拓扑 | 固定 RPC 调用链 | 动态 Agent 循环，轮次/分支/工具不确定 |
| 数据敏感 | 用户 ID / 订单号 | 完整 Prompt / Completion 可能含 PII / 业务秘密 |
| 重放 | SQL 重跑 | 需要完整 Prompt + 模型版本 + 温度 + 种子 |
| 资源瓶颈 | CPU / DB 连接 | GPU SM 利用、HBM、KV cache、网络带宽 |
| 回归 | 单测 + 集成测试 | 数据集 + LLM-as-Judge + 人工标注 |

**LLM 系统需要新信号、新存储、新评估闭环**。Llm Observability 4 Layer Quant67#^Block 3

## 核心指标

### 延迟

- **TTFT** — Time To First Token；Prefill + 排队决定；用户主观"响应快不快"
- **TPOT / ITL** — Time Per Output Token / Inter-Token Latency；Decode + 显存带宽
- **E2E** = TTFT + TPOT × output_tokens
- **Queue Time** — 高并发时吞没一切优化

告警门槛（13B/32B 模型）：

- TTFT p95 < 500 ms（对话式）/ < 2 s（长文档/Agent）
- TPOT p95 < 50 ms（≥ 20 tokens/s）
- Queue p99 < 1 s

### 吞吐

- `output_tokens_per_second` — 单卡总输出，最贴近"成本效率"
- `requests_per_second` — 对比同一模型不同后端
- **`goodput`** — 同时满足 SLO（TTFT、TPOT 都达标）的 throughput，业界新共识指标

### GPU 与引擎内部信号

- **SM 利用率** — 不是 `nvidia-smi` 那个 100%（只是"有 kernel 在跑"），要看 DCGM 的 `DCGM_FI_PROF_SM_ACTIVE` / `DCGM_FI_PROF_PIPE_TENSOR_ACTIVE`
- **HBM 占用** — `DCGM_FI_DEV_FB_USED`，超 90% 意味着 KV cache 即将 swap
- **KV Cache 使用率** — vLLM `vllm:gpu_cache_usage_perc`
- **Prefix Cache 命中率** — SGLang `sglang:cache_hit_rate` / vLLM automatic prefix caching；**命中 60% 以上 TTFT 能砍一半**
- **Running / Waiting 请求数** — 队列深度，扩容信号

### Token 成本

| 类型 | 相对价格 | 典型应用 |
|---|---|---|
| Input (uncached) | 1× | 新 prompt |
| Cached input | 0.1× ~ 0.5× | 系统提示 / 长文档重用（OpenAI / DeepSeek / Anthropic 均支持） |
| Output | 3× ~ 5× | 生成 token |

观测侧分别记录 `prompt_tokens` / `completion_tokens` / `cached_tokens` / `reasoning_tokens`（o1/R1）。**缓存命中率 = cached_tokens / prompt_tokens**；成熟 RAG 系统应稳定 50%+。

### 质量

- **满意度** — 显式 👍/👎（LangSmith、Langfuse SDK 回写）；隐式信号包括"是否复制"、"是否追问"、"是否停止生成"
- **引用覆盖率**（RAG）
- **Refusal Rate** — 太低可能越权，太高体验差
- **Answer Length 分布** — 突变预示 prompt 被污染
- **Groundedness / Faithfulness** — RAGAS 打分 0–1

## Trace 标准

### OpenTelemetry GenAI Semantic Conventions

OTel 在 2024 底到 2025 陆续把 `gen_ai.*` 语义约定从 experimental 推向 **stable**。关键字段：

```
gen_ai.system                = "openai" | "anthropic" | "deepseek" | ...
gen_ai.operation.name        = "chat" | "text_completion" | "embeddings"
gen_ai.request.model         = "gpt-4o-mini"
gen_ai.usage.input_tokens    = 1234
gen_ai.usage.output_tokens   = 456
gen_ai.response.finish_reasons = ["stop"]
```

工具调用 / Agent 扩展字段：`gen_ai.tool.name` / `gen_ai.tool.call.id` / `gen_ai.agent.id` / `gen_ai.server.request.duration` (histogram) / `gen_ai.client.token.usage` (histogram)。

**原则**：默认不把 prompt / completion 塞进 attribute（PII），用 Events 承载；`OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT` 环境变量控制是否采集内容。Llm Observability 4 Layer Quant67#^Block 5

### OpenInference（Arize）与 OpenLLMetry（Traceloop）

两套早于 OTel 稳定版的"事实标准"：

- **OpenInference** — Arize Phoenix 推动，`openinference.span.kind = LLM/CHAIN/RETRIEVER/RERANKER/EMBEDDING/TOOL/AGENT`；已有向 OTel GenAI 对齐的适配
- **OpenLLMetry** — Traceloop 推动，`traceloop-sdk` 对 20+ 框架（LangChain、LlamaIndex、Haystack、Mistral、Bedrock…）做 monkey-patch instrumentation；一行 `Traceloop.init()` 即可接管

**2025 实际姿势**：后端接收 OTel，SDK 任选。Langfuse、Phoenix、SigNoz、Jaeger、Tempo、Dynatrace 都开始原生理解 `gen_ai.*`。

### Span 层次模板

```
Trace: "user chat #U123"
└─ Span: agent.run                           (AGENT)
  ├─ Span: retriever.search                 (RETRIEVER)
  │  └─ Span: embedding.encode              (EMBEDDING)
  ├─ Span: reranker.rerank                  (RERANKER)
  ├─ Span: llm.plan (gpt-4o-mini)           (LLM)
  ├─ Span: tool.sql_query                   (TOOL)
  ├─ Span: tool.http_fetch                  (TOOL)
  └─ Span: llm.summarize (claude-3-5)       (LLM)
```

每个 LLM span 都带：model / params / usage / cost / TTFT / TPOT / finish_reason / messages（可脱敏）。

## 主流通用栈横评

| 工具 | 模式 | 特色 | 缺点 |
|---|---|---|---|
| **LangSmith** | 商业 | Playground / Dataset & Eval / Annotation Queue；深度绑定 LangChain/LangGraph | SaaS 为主，数据出境 |
| **Langfuse** | 开源 self-host | ClickHouse + Postgres；OTel ingest；Prompt 版本化；国内可 self-host；Eval 内置 LLM-as-Judge + RAGAS/DeepEval 桥接 | 需自运维 |
| **Helicone** | 开源 + SaaS | 代理式接入（`base_url` 改 `oai.helicone.ai` 零代码） | 代理模式引入一跳延迟；高级 Eval 弱 |
| **Arize Phoenix** | 开源 | OpenInference 参考实现；离线调试（`px.launch_app()`）；内置 RAG 三角形可视化 | 偏 offline |
| **W&B Weave** | SaaS + 自托管 | 与 W&B 训练/评估平台打通 | 与训练强绑定 |
| **OpenLLMetry / Traceloop** | 开源 SDK | 自动 instrument 20+ 框架；纯标准 OTel，后端随便选 | 仅是 SDK，无后端 |
| **AgentOps** | SaaS | 专攻 Agent：轮次 / 工具成功率 / Session Replay；与 CrewAI / AutoGen / LangGraph 集成 | 偏 Agent 场景 |
| **Pezzo** | 开源 | Prompt 作为一等公民（版本/环境/A/B） | 功能较窄 |
| **Lunary** | 开源 + SaaS | 原 LLMonitor，偏产品化；带用户管理、分析面板、Eval | 已停止部分维护 |

### 选型决策树

```
我要观测什么?
├─ 数据能出境?
│  ├─ 能 → LangSmith / Helicone / Lunary
│  └─ 不能 → Langfuse / Phoenix self-host
├─ 已有 OTel/APM?
│  ├─ 是 → OpenLLMetry 发到现有后端
│  └─ 否 → Langfuse / Phoenix self-host
├─ 重点是 Agent?
│  └─ 是 → AgentOps + Langfuse
└─ 重 Prompt 版本?
   └─ 是 → Pezzo / Langfuse / LangSmith
```

## 评估闭环

### 在线 vs 离线

```
Prod → 采样 1% → OnlineEval (LLM-as-Judge) → 仪表盘 / 告警
Prod → badcase → 标注队列 → Dataset → 离线 Eval / CI → 发布闸门
```

### LLM-as-Judge 维度

Relevance / Groundedness / Faithfulness / Helpfulness / Toxicity / Conciseness。

**关键规则**：裁判模型必须比生产模型更强或至少同级，否则存在 judge 偏见；定期和人工标注对齐。

常见坑：

- 只用一个 judge → bias 放大 → "pairwise 对比 + 多 judge 投票"
- judge 本身也会幻觉 → 要求"只对能在 context 中找到依据的 claim 给高分"
- 裁判 prompt 过短 → 加 few-shot + 严格 JSON schema
- 成本失控 → 只在"模型回答置信度低"或"用户点踩"时触发

### 离线评估框架

- **RAGAS** — RAG 专用，faithfulness / answer_relevancy / context_precision/recall
- **DeepEval** — pytest-like，`assert_test(GEval(...))`，CI 友好
- **Giskard** — 自动扫描（bias / prompt injection / robustness）
- **promptfoo** — YAML 驱动矩阵评估，适合 prompt 工程师 CLI 流
- **Arize Phoenix Evals** — 与 trace 打通

### 数据集治理

- **冻结集（Golden Set）** — 人工标注、代表性强，不再扩充，用于跨版本对比
- **滚动集** — 从线上 badcase 不断采样，用于回归
- **对抗集** — 安全 / 越狱 / prompt 注入样本
- **每次 Prompt 或模型变更，两类数据集分数都不能回退才允许发布**

## 幻觉的 3 种形态

工程上建议把"幻觉"细分，不同形态处置不同：

- **事实型幻觉** — 捏造人物、日期、数字。→ Groundedness + 外部知识核查
- **引用型幻觉** — 引了存在的文档但原文不支持结论。→ claim 级 entailment 检查
- **指令幻觉** — 不按用户要求的格式 / 约束输出。→ 结构化输出（JSON Schema / function call）+ 校验重试

**不同形态的观测指标、告警阈值和修复手段都不一样**，把它们混在一个"hallucination rate"里会误导方向。

## Agent 观测

### 特有信号

- **轮次（steps）** — thought-action-observation 次数
- **工具成功率** — 分 HTTP 错 / 参数错 / 语义错
- **卡死 / 环** — 连续 N 步调用同一工具同一参数
- **计划偏移** — Planner 输出 plan 和实际 action 序列相似度
- **成本/轮次** — Agent session 总 token / 总工具费

### 死循环检测

```python
def loop_detector(history, window=4):
    sig = [(a.tool, hash(str(a.args))) for a in history[-window*2:]]
    half = len(sig) // 2
    return sig[:half] == sig[half:]  # 后半和前半完全相同 -> 疑似环
```

检测到立刻打断并记 `agent.loop_detected=true`。

### 多 Agent 拓扑

- Agent 间"消息图"（谁发给谁、哪一步触发）
- 每个 Agent 的子 LLM 成本与延迟归属
- 角色划分是否按预期生效（例如 "critic" 是否真的在打分）

LangGraph 的 state machine 天然适合 trace 成一棵树，Langfuse 的 session 视图能覆盖；复杂场景下 AgentOps + 自定义 dashboard 更清晰。

## 成本观测

仪表盘必看 3 类缓存命中率：

- **Prefix cache hit rate**（引擎侧，vLLM/SGLang）
- **Prompt cache hit rate**（API 侧，OpenAI / Anthropic / DeepSeek）
- **Semantic cache hit rate**（网关侧，GPTCache 等）

三者独立，都能省钱；目标：总 input cost 下降 30–60%。

## 日志合规

- **EU GDPR** — 个人数据"收集最小化"、"目的限定"、"可被遗忘"；数据泄露 72 小时上报
- **中国《个人信息保护法》** — 敏感个人信息需单独同意；跨境传输需安全评估
- **中国《生成式人工智能服务管理暂行办法》**（2023.8 施行）— 保存用户输入、模型输出日志**至少 6 个月**；内容安全审核义务；训练数据合法性义务
- **美国行业法** — HIPAA（医疗）/ FINRA（金融）/ FERPA（教育）

工程落地：日志分级（结构化永久留 / prompt-completion 按法规 TTL）/ 数据分区 / 审计日志 / "可遗忘"（按 `user_id` 级联删除）/ 安全审核记录单独入审计库。

## 国内外厂商生态

| 厂商 | 集成方式 | 限制 |
|---|---|---|
| OpenAI | Platform Dashboard + Responses API `trace_id` | 出口 |
| Anthropic | Console prompt caching 命中 + `cache_control` | 出口 |
| Vertex AI | Cloud Monitoring + `gen_ai.*` OTel 语义 | 出口 |
| AWS Bedrock | CloudWatch Metrics + Model Invocation Logging | 出口 |
| 阿里云 PAI / 百炼 | "模型观测"模块 + SLS 打通 | 境内 |
| 火山引擎方舟 | AppMetrics / TCE 体系 + OTel 导出 | 境内 |
| DeepSeek / Kimi / 智谱 / 百川 | 仅 Dashboard 展示 usage | — |

**结论**：厂商 Dashboard 看"账单和健康"够用，真要排障还是得在应用侧独立上一套 trace。

## 典型排障故事（prefix 失效风暴）

> 故事："周一早上客服机器人变慢"

1. **Grafana 红灯**：TTFT p95 从 0.6 s 冲到 3.5 s，error_rate 平稳 — 不是"模型返回错"而是"慢"
2. **拆分维度**：按 model 看只有 `qwen3-32b` 变差；按 tenant 看 `tenant-A` 占突增 80%；按 region 集中在单机房
3. **引擎指标**：`vllm:num_requests_waiting` 堆积、`gpu_cache_usage_perc` 96%、`prefix_cache_hit_rate` 从 55% 掉到 8%、`DCGM_FI_PROF_SM_ACTIVE` 仍然很高 — **不是 GPU 闲着，而是在"徒劳"地重复 prefill**
4. **结论**：tenant-A 换了新 system prompt，prefix 变了导致缓存全失效，带来 prefill 风暴，KV cache 被挤爆
5. **修复**：通过 LLM 网关的 prompt 模板统一化，把变动的部分挪到尾部，恢复 prefix 复用
6. **复盘**：给 Prompt 变更加上"prefix cache hit 回归门槛" — 离线 eval 跑 1000 条典型对话，命中率低于基线 10% 不允许发布；告警里加上 `prefix_cache_hit_rate` 指标

**每一步都依赖不同层的可观测性**：业务层告警 → 基础设施层指标 → 调用层 trace → Prompt 版本系统。**没有任一层都排不出来** — 这就是"四层观测模型"的价值。

## 与训练侧观测的对比

| 关注点 | 训练侧 | 在线推理 / 应用侧 |
|---|---|---|
| 迭代周期 | 一个 job 几天~几周 | 每秒数百请求 |
| 关键指标 | loss / gradient norm / MFU / TFLOPS / GPU 健康 | TTFT / TPOT / QPS / cost / quality |
| 工具 | W&B / TensorBoard / MLflow / SwanLab | Langfuse / LangSmith / Phoenix / APM |
| 失败模式 | 发散 / NaN / 节点坏 / 通信卡顿 | 幻觉 / 越权 / 成本爆炸 / 延迟尖刺 |
| 数据产物 | Checkpoint / log | Trace / Prompt 版本 / Eval score |

两侧共用基础设施：GPU 遥测（DCGM）/ 分布式 tracing（OTel）/ 告警总线（Alertmanager）。

## 前沿方向

- **AIOps for AI** — 用 LLM 看 trace 直接给工程师人话根因结论；Arize、Dynatrace、Datadog 都在试；Langfuse "trace summary" beta
- **Trace + Weights 联合剖析** — 训练框架（Megatron、Mcore）把 step 级 loss / grad norm 与 trace 对齐，定位"是训练分布问题还是推理工程问题"

## 相关实体

- [[concepts/agent-evaluation-benchmark-frameworks]] — 评测框架
- [[concepts/agent-security-architecture]] — Agent 安全
- [[entities/amap-abot-earth-0.5-3d-native-world-model]] — 3D world model
- [[raw/articles/llm-observability-4-layer-quant67|原文存档]]

## 所属 MOC

- [[moc/layer-5-production-security|Layer 5 Production Security]]
