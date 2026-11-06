---
title: "Apple Foundation Models"
created: 2026-06-16
updated: 2026-11-07
type: entity
tags: [article, newsletter]
source_url: "https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/apple-foundation-models"
sources: [raw/articles/anthropic-apple-foundation-models-claude]
review_value: 7
review_confidence: 7
review_recommendation: strong
review_stars: 4
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Apple Foundation Models

> Source: [[raw/articles/anthropic-apple-foundation-models-claude|原文存档]] ^[raw/articles/anthropic-apple-foundation-models-claude.md]

## 核心要点

- **来源**: https://platform.claude.com/docs/en/cli-sdks-libraries/libraries/apple-foundation-models
- **评分**: v=7, c=7, v×c=49, stars=4
- **评估理由**: Solid technical documentation for integrating Claude with Apple's Foundation Models framework via a Swift package. Well-structured with clear sections on installation, quick start, model selection, effort levels, and authentication (dev vs production). Authoritative source from Anthropic. Notable do

## 内容提炼

CLI, SDKs, and libraries Libraries and integrations ^[raw/articles/anthropic-apple-foundation-models-claude.md]

Use Claude on Apple platforms through the Foundation Models framework with the Claude for Foundation Models Swift package. ^[raw/articles/anthropic-apple-foundation-models-claude.md]

[Claude for Foundation Models](https://github.com/anthropics/ClaudeForFoundationModels) is a Swift package that makes Claude available as a server-side language model in Apple's [Foundation Models](https://developer.apple.com/documentation/foundationmodels) framework. The package conforms Claude to the framework's `LanguageModel` protocol, so you drive it with the same `LanguageModelSession` API you use for Apple's on-device model: `respond(to:)`, streaming, guided generation, and tool calling all work the same way. ^[raw/articles/anthropic-apple-foundation-models-claude.md]

Requests go directly from your app to the Claude API; Apple is not in the request path and does not see prompts or responses. Usage is billed to your Anthropic account at [standard API pricing](https://platform.claude.com/docs/en/about-claude/pricing). Your app decides when to use Claude and when to use Apple's on-device model: pass whichever model you want to each session. ^[raw/articles/anthropic-apple-foundation-models-claude.md]

**Beta.** This package targets the Foundation Models server-side language model API introduced in the OS ^[raw/articles/anthropic-apple-foundation-models-claude.md]

## 关键洞察

- Beta.** This package targets the Foundation Models server-side language model API introduced in the OS 27 betas. APIs may change before general availability.
- iOS 27, macOS 27, visionOS 27, or watchOS 27 (all in beta): the OS releases whose Foundation Models framework supports server-side language models
- ### When to use Claude versus the on-device model
- Prompt caching controls (the package applies prompt caching automatically; cache TTL and breakpoint placement are not configurable)

## 实践启示

- 文章的核心论点可在生产环境验证
- 与现有实体的差异化角度：本文来自 platform.claude.com 视角
- 引用源：[[raw/articles/anthropic-apple-foundation-models-claude]] ^[raw/articles/anthropic-apple-foundation-models-claude.md]

## 深度分析

### 架构本质：协议适配器，而非普通客户端 SDK

Claude for Foundation Models 不是把 Messages API 照搬进 Swift，而是让 Claude「扮演」Apple 设备端模型：package 将 Claude 适配为 Foundation Models 框架中的 server-side language model，实现框架的 `LanguageModel` 协议，于是 `respond(to:)`、流式、guided generation、工具调用全部复用设备端同一套 `LanguageModelSession` API ^[raw/articles/anthropic-apple-foundation-models-claude.md:17]。切换的成本被压到「换一个 `model:` 参数」——应用按任务自行决定何时用 Claude、何时用端上模型 ^[raw/articles/anthropic-apple-foundation-models-claude.md:19]。公开面也刻意收窄：只暴露 provider 适配与配置类型（`ClaudeLanguageModel`、`ClaudeModel`、`AuthMode`、`ClaudeServerTool`），不提供通用 Messages API 客户端 ^[raw/articles/anthropic-apple-foundation-models-claude.md:23]。这决定了它的定位：Apple 生态内的推理插槽，而非全功能 SDK。

### 信任模型：Apple 出局、能力显式声明、计费直连

数据路径是这次集成的核心卖点：请求由应用直达 Claude API，Apple 不在请求路径上、看不到提示词与响应，费用按标准 API 价格计入 Anthropic 账户 ^[raw/articles/anthropic-apple-foundation-models-claude.md:19]。能力协商则用「显式声明 + 硬失败」替代猜测：每个 `ClaudeModel` 常量声明自己接受的采样参数、effort、adaptive thinking、结构化输出与图像输入；编译进包的新模型随版本发布，未收录的 ID 必须手工声明能力，且**刻意不设猜测型简写**——因为向模型发送其不支持的字段是硬错误 ^[raw/articles/anthropic-apple-foundation-models-claude.md:55]。Beta 状态同样明示：包面向 OS 27 beta 引入的 API，GA 前可能变动 ^[raw/articles/anthropic-apple-foundation-models-claude.md:21]。

### 请求面：effort 锁定、服务端工具、错误映射的三处设计取舍

effort 等级用 `fixedEffort:` 逐请求锁定，优先级高于框架的 per-request reasoning hints；`.xhigh`/`.max` 只有这条路可达，因为框架推理档位止步于 high ^[raw/articles/anthropic-apple-foundation-models-claude.md:59]。服务端工具（web search、web fetch、code execution）在 Anthropic 基础设施上单轮内完成，配置挂在 `ClaudeLanguageModel` 而非 session 上——因为 session 类型属于 Apple，要按会话换工具集就得构造多个实例 ^[raw/articles/anthropic-apple-foundation-models-claude.md:101-105]。错误映射走语义对齐：context 溢出→`.contextSizeExceeded`、HTTP 429→`.rateLimited`、超时→`.timeout`，无对应框架错误的原样抛 `ClaudeError`；典型模式是捕获 `.rateLimited` 后降级到 `SystemLanguageModel`、排队或给出重试入口 ^[raw/articles/anthropic-apple-foundation-models-claude.md:113-115]。

### 边界清单：协议表达不了的能力与不可配置项

协议表达不了的功能直接缺席，包括 prompt caching 控制（包自动应用，TTL 与断点不可配置）、stop sequences、批处理、Files API、token 计数、beta headers ^[raw/articles/anthropic-apple-foundation-models-claude.md:119-126]。认证分两段：开发用 `.apiKey` 直连，生产必须走 `.proxied` 后端代理补 `x-api-key`——文档明言打包进应用的 key 可从发布二进制中提取 ^[raw/articles/anthropic-apple-foundation-models-claude.md:75-81]。外包一层环境约束：需 iOS/macOS/visionOS/watchOS 27 beta 与 Xcode 27 beta ^[raw/articles/anthropic-apple-foundation-models-claude.md:27-28]。

1. 把「协议适配器」当默认架构直觉：当目标平台已有模型抽象层（此处是 `LanguageModel` 协议）时，优先让第三方模型 conform 它，而非另起一套平行 API——切换成本几乎归零 ^[raw/articles/anthropic-apple-foundation-models-claude.md:17]
2. 云端/端上混合推理按任务路由：端上模型负责快、私有、离线的轻任务，需要更大上下文、前沿推理或 web search/code execution 时升级到 Claude ^[raw/articles/anthropic-apple-foundation-models-claude.md:65]
3. 生产环境绝不内嵌 API key：开发用 `.apiKey`，上线切 `.proxied` 代理在服务端补凭证；这条同样适用于其他云模型 SDK ^[raw/articles/anthropic-apple-foundation-models-claude.md:75-79]
4. 能力矩阵显式声明、硬失败优于静默降级：发送不支持字段是硬错误，所以包拒绝猜测能力、结构化输出不支持时抛 `unsupportedGenerationGuide` ^[raw/articles/anthropic-apple-foundation-models-claude.md:55,91]
5. 错误处理即产品策略：`.rateLimited` 捕获后降级端上模型、排队或重试，是混合推理应用的标配容错 ^[raw/articles/anthropic-apple-foundation-models-claude.md:115]
6. Beta 依赖要写进发布决策：目标 API 处于 OS 27 beta，GA 前可能变动，加上 beta 期不接受外部 PR，升级节奏应绑定上游包版本而非自行 fork ^[raw/articles/anthropic-apple-foundation-models-claude.md:21,136]
## 相关实体
- [[entities/from-doer-to-director-the-ai-mindset-shift|from doer to director: the ai mindset shift]]
- [[entities/why-internally-built-ai-fails-fund-accounting-audits|why internally-built ai fails fund accounting audits]]
- [[entities/back-up-and-restore-your-amazon-eks-cluster-resources-using-velero-amazon-web-se|back up and restore your amazon eks cluster resources using ]]


