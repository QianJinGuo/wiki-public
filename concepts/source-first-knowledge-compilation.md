---
title: "source-first 知识编译"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, source-first, provenance, knowledge-base, llm-wiki, trust]
sources: [entities/hermes-wiki-9-step-auto-growing-knowledge-network]
---

## 定义

source-first 知识编译要求：每条进入知识库的结论必须绑定源头（脚注引用 raw 目录下的具体文件），AI 生成的归纳必须可追溯回原文。核心信条「raw 是证据层，源头坏了后面全乱」——没有 [[entities/agent-memory-storage-six-schools-wiki-compile-vs-raw-data-debate|来源信息的分层]] 的 AI 知识库长期看就是不可信的资料堆。

## 核心范式

- **raw/ 只追加只读**：原始资料一旦入库不修改，保证源头唯一性
- **结论绑定脚注**：每段关键归纳用 caret-脚注标注 raw 路径出处
- **AI 归纳明确标记**：模型推断 vs 原文结论必须显式区分
- **lint 校验**：自动检测无 provenance 的高价值段落，强制补源

## 背景与提出

source-first 知识编译的提出背景是 AI 知识管理的核心可信度危机：AI 知识库里的内容，有多少是原文，有多少是 AI 的归纳/推断/幻觉？当用户基于知识库的一条结论做决策时，这条结论的可信度谁来保证？如果知识库的源头不清晰，整个系统就建立在沙子上。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

这个问题的严重性在 2025 年开始被广泛认识到：随着 LLM 知识库产品（Notion AI、Confluence AI、Obsidian AI）的普及，大量用户开始把 AI 生成的总结当作「知识」来使用。但这些总结和原始资料之间的映射关系是模糊的——用户很难判断「这句话是原文说的，还是 AI 推断的」。Source-first 范式是对这个问题的直接回应：在知识编译阶段就明确标注每条结论的来源，让 provenance 成为一等公民。 ^[entities/llm-wiki-architecture]

[[entities/llm-wiki-architecture-karpathy-markdown-knowledge-base|Karpathy 的 LLM Wiki]] 和 [[entities/hermes-skills-llm-wiki-self-improving-knowledge-system|自举式 wiki 知识系统]] 进一步揭示了另一个维度：当知识库本身在自我生长时，每一轮 AI 生成的「元知识」（关于知识质量的判断、关于矛盾信息的标记）也需要有 provenance，否则知识网络的自我纠错机制本身就成了新的幻觉来源。

## 范式细节

raw/ 只追加只读——原始资料一旦入库不修改，保证源头唯一性。这个约束的核心价值是「可审计性」：当知识库中的某个结论被质疑时，可以回溯到 raw/ 中的原始资料进行核实。如果 raw/ 可以被修改，就无法保证「知识库的结论和原始资料一致」这个基本承诺。^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

结论绑定脚注——每段关键归纳用 caret-脚注标注 raw 路径出处。这个脚注格式（caret + 方括号 + raw 路径 + 方括号）使得任何结论都可以被追溯到原始文档的具体位置。当多个来源对同一结论有不同描述时，脚注可以并列多个来源，让读者看到「关于这件事，有 A 说、B 说、C 说」，而不是只有一个 AI 的综合版本。[[entities/amazon-quick-research-agentic-multi-source-citation|多来源引用的研究 Agent]] 在这方面提供了技术参照：它通过并行爬取+多源综合来保证结论的 diversity，但 source-first 范式走的是另一条路——不是在综合时覆盖矛盾，而是让矛盾显式存在于脚注层级。

AI 归纳明确标记——模型推断 vs 原文结论必须显式区分。这个区分的价值在于：原文结论是「事实锚点」，模型推断是「可能正确的推测」——两者可信度完全不同。标记清楚之后，用户可以自己判断一条 AI 推断是否值得采信，而不会把它当成原文的直接引用。 ^[entities/llm-wiki-architecture]

lint 校验——自动检测无 provenance 的高价值段落，强制补源。这个机制是 source-first 范式能够真正落地的关键：不是靠人的自觉，而是靠系统强制。Hermes-Wiki 的 SCHEMA.md 中要求「关键结论尽量绑定来源」「不确定内容必须标记为待验证」——这些规则通过 lint 自动执行，而非依赖人工审核。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

[[entities/knowledge-mgmt-is-moat|知识管理作为护城河]] 的核心论点与此呼应：知识库的可信度是长期积累的结果，而 lint 强制机制确保了「可信度」这个属性不会随时间衰减——因为任何绕过 provenance 的尝试都会被自动检测拦截。

## 局限与反对声音

第一个局限是「来源信息的损耗」：一段 50 页的报告被编译成 5 个 wiki 页面，每个页面若干个脚注引用原始文档。但原始报告中的上下文、论证链条、数据来源的细致分层，在编译过程中已经被大幅压缩了。即使有脚注指向原文，读者也无法仅凭脚注重建原始论证的完整结构。这说明 source-first 范式只能保证「结论有来源」，不能保证「来源的上下文被完整传递」。

第二个局限是「来源质量的分层」：raw/ 里的原始资料本身质量参差不齐——权威论文和社交媒体帖子的可信度完全不同，但 source-first 范式中它们只是「有来源」而已，没有对来源本身的质量做出判断。V2 的置信度评分机制尝试解决这个问题，但置信度评分本身是由 AI 判定的，AI 对来源质量的判断也可能是错的。

第三个局限是「编译工作量的增加」：要求每条结论绑定脚注，实际上是把「标注工作」从「可选」变成「必须」。这在短期内显著增加了知识编译的成本——每个 Agent 编译动作不再只是「生成结论」，还要「标注来源」「区分推断和原文」「标记待验证」。对于大量资料需要快速入库的场景，这个成本可能是不可接受的。

## 现实案例

本 wiki 的 Hermes-Wiki 9 步法是 source-first 知识编译的完整工程实现：步骤 6 收录第一篇文章时明确要求「关键结论必须标注来源：raw/articles/你的文章标题.md」「如果某些内容是总结归纳或待验证判断，请明确标记」，步骤 7 的 Obsidian 5 点验收中「log.md」要求记录「来源、待验证内容」。三件套（SCHEMA + log + lint）共同保证了每条进入知识库的知识都有清晰的 provenance。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

Hermes-Wiki 的 log.md 是 source-first 范式的时间维度体现：每次新建/更新页面、标注来源、标记待验证项，都记进 log.md。没有 log，AI 知识库就是黑箱——页面多了你不知道哪来的、哪些是原文结论、哪些是 AI 归纳。log.md 把知识库的变更历史完整记录下来，使得「知识是怎么来的」这个问题在任何时候都可以被回答。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

LLM Wiki 的 Semantic File System + Context Graph 架构是企业级 source-first 实现的参考：每个知识节点记录「这条结论出自哪场会议」「谁是当前的 Owner」「信息的保鲜期过了吗」「置信度有多高」「权限边界在哪里」——这比简单的文件路径脚注提供了更丰富的 provenance 元数据。Context Graph 将业务 Artifact 的关系固化，使得元数据的变更（如「负责人变更」「决议被新决议覆盖」）能够触发关系网络的更新传播。 ^[entities/enterprise-ai-memory-substrate-three-layer-architecture]

[[entities/rag-vs-llm-wiki-enterprise-knowledge-base|RAG vs LLM Wiki 企业知识库对比]] 分析了另一种知识管理范式：RAG 以检索为中心，通过向量相似性找到相关段落再塞入 context；LLM Wiki 以编译为中心，通过 AI 主动组织知识网络。两者在 provenance 处理上的哲学差异显著——RAG 承认「知识在外部」，只负责检索；Wiki 认为「知识需要内化」，通过编译过程将外部知识转化为内部可验证的网络节点。

## 深度分析

### source-first 与知识管理范式转移

Source-first 范式折射出更宏观的知识管理范式转移：从「信息检索」到「知识编译」，从「以文档为中心」到「以结论为中心」。传统知识管理（Wiki 1.0、Confluence）以文档为原子单位，知识组织依赖标题层级和标签系统——这对于人类阅读是自然的，但对于 AI 理解知识网络内部的依赖关系并不高效。Source-first 编译以「结论 + 出处」为原子单位，每个结论节点天然携带指向源头的有向边，这使得知识网络可以用图结构而非树结构来组织。[[entities/knowledge-mgmt-is-moat|知识作为护城河]] 的论述进一步指出，在 AI Native 时代，这种可验证的知识组织方式将成为组织认知资产的核心竞争力。

### provenance 的技术实现路径

Provenance 的技术实现有三种主流路径：文件路径脚注（Hermes-Wiki 的 caret 格式）、多模态元数据标签（LLM Wiki 的 Context Graph）、以及结构化溯源协议（OpenProve 系的 W3C Provenance 标准）。Hermes-Wiki 选择文件路径脚注是因为它最简单、最符合 Markdown 的天然语序——不需要额外的 JSON-LD 头部，不需要特殊的渲染插件，普通编辑器就能写、普通阅读器就能看。缺点是路径本身不携带「这条路径指向的内容是否还存在」「原始文档的版本是什么」这类动态信息。[[entities/graphify-software-engineering-knowledge-graph|知识图谱]] 的引入可以在一定程度上弥补这个缺陷——节点携带版本指纹，边携带时间戳，知识网络本身变成可验证的分布式账本。

### source-first 的规模化瓶颈与解决方向

Source-first 范式在大规模知识编译时会遇到三个瓶颈：来源标注的标准化问题（不同编译者的「关键结论」判断标准不一）、跨文档交叉引用的一致性问题（文档 A 引用文档 B 的某个具体段落，B 被更新后 A 的脚注是否仍然有效）、以及多语言来源的处理问题（英文论文的某个 claim 能否对应到中文报告的某个段落）。解决方向包括：制定「关键结论判定指南」（类似法律上的「重大事实」标准）、引入 [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|自动化评估框架]] 来校准不同编译者的标注质量、以及建立来源等效性映射层（cross-document claim alignment）。这些方向都是 source-first 范式在 2026 年之后的活跃研究领域。

## 进一步阅读

- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki 9 步法]]
- [[entities/llm-wiki-architecture|LLM Wiki 架构]]
- [[concepts/llm-wiki-paradigm|LLM Wiki 范式]]
- [[concepts/memory-source-provenance|Agent 记忆 provenance]]
- [[concepts/knowledge-network-self-growth|知识网络自生长]]
- AI pipeline 数据质量

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
