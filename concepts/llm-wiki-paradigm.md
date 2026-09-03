---
title: "LLM Wiki 知识范式"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, llm-wiki, karpathy, knowledge-base, obsidian, paradigm]
sources: [entities/karpathy-llm-wiki-v2-2026, entities/karpathy-llm-wiki-second-brain-awkthole, entities/llm-wiki-architecture]
---

## 定义

LLM Wiki 是 Karpathy 提出的知识管理范式：用 wiki 结构作为人和 LLM 共享的「第二大脑」，每条知识有明确路径、可被 wikilink 引用、可被 agent 编译为新的关联节点。从「丢资料 + 全文搜」升级为「结构化网络 + 关联推理」。

## 核心范式

- **wiki 是 substrate**：人和 LLM 共用同一份结构化笔记，不是各存一份
- **wikilink 是网络底层**：没有双链，wiki 就退化为文件夹
- **LLM 作为编译器**：把 raw 资料拆成 concept / entity / MOC / draft 节点
- **网络效应非线性**：节点越多、互链越密，新增节点的价值放大越快

## 背景与提出

Karpathy 在 2025 年 5 月发布了他的 LLM Wiki v2 方案，这个方案的核心动机是对 RAG 范式的根本性质疑：「大多数人在用 AI 处理文档的方式，本质上都是 RAG——不管你知不知道这个词。」RAG 每次都是临时检索、临时回答，知识的沉淀程度止步于「能被检索到」，但页面之间没有建立关联、没有交叉引用、没有随新信息更新而进化的能力。 ^[entities/karpathy-llm-wiki-v2-2026]

这个问题的具体场景是：拿到一篇几十页的行业报告，丢给 ChatGPT 聊了二十分钟，观点梳理得清清楚楚。第二天想引用其中一个数据点，打开新的对话窗口，重新上传报告，重新提问——一切从头来过。上周聊过的那个精彩洞察？没了。上个月让 AI 帮你对比过的那三篇论文？也得重新来。LLM Wiki 提出的解决方案是：与其让 AI 每次临时翻书找答案，不如让它像一位全职的百科全书编辑，持续地阅读你喂给它的资料，提炼、整合、交叉引用，维护一部专属于你的知识库。 ^[entities/karpathy-llm-wiki-v2-2026]

LLM Wiki 发布不到 48 小时，GitHub 星标破了 5000。这说明「知识无法积累」是大量 AI 用户的共同痛点，而 Karpathy 的方案（让 AI 承担维护者的角色）击中了真实的需求。 ^[entities/karpathy-llm-wiki-v2-2026]

## 范式细节

wiki 是 substrate 这个原则具体展开：人和 LLM 共用同一份结构化笔记，不是各存一份。这意味着 wiki 的每一个页面都是 LLM 的「工作记忆」的一部分——当你打开一个 wiki 页面，LLM 知道这个页面的内容，也知道这个页面链接到哪些其他页面，这些链接关系本身就是上下文的一部分。

wikilink 是网络底层：没有双链，wiki 就退化为文件夹。Obsidian 的 wikilink 语法（双层方括号）同时建立前向链接和反向链接，使得被引用节点能感知所有引用者。这种双向感知能力是单向 hyperlink 无法提供的——hyperlink 只能正向跳转，无法回答「这个概念被哪些其他概念谈到」这个问题。 ^[entities/obsidian]

LLM 作为编译器：把 raw 资料拆成 concept / entity / MOC / draft 节点。这个「编译」过程的本质是信息压缩和结构化——原始资料是低结构的文本，wiki 页面是高结构的节点网络。压缩过程中会丢失细节，但换来的是关联性和可检索性。V2 引入了置信度分数（根据来源权威性、独立来源数量、时间衰减三个因素动态调整），试图在「压缩」和「保真」之间找到更精细的平衡。 ^[entities/karpathy-llm-wiki-v2-2026]

网络效应非线性：节点越多、互链越密，新增节点的价值放大越快。这与梅特卡夫定律（网络价值与节点数的平方成正比）同构——当 wiki 只有 10 个节点时，新增第 11 个节点的关联路径有限；当 wiki 有 1000 个节点时，新增第 1001 个节点可以和已有网络中的大量节点建立关联，其价值被显著放大。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

## 局限与反对声音

第一个局限是「摘要漂移」：AI 对同一份资料进行多轮改写后，内容会逐渐偏离原始材料。这和人类记忆的「重构性」类似——每次回忆都会微妙地改变记忆内容。V2 引入的来源追踪（每条知识标注来源）和定期「健康检查」（检测自相矛盾的页面）是对这个问题的缓解，不是根治。 ^[entities/karpathy-llm-wiki-v2-2026]

第二个局限是「维护负担的转嫁而非消除」：Karpathy 的核心主张是「让 AI 承担维护工作」，但 AI 的维护能力受限于模型本身的理解深度——当 wiki 规模增长到跨多领域时，单一模型无法对所有领域的知识质量做出可靠判断，专业领域的知识验证仍然需要人类专家参与。

第三个局限是「RAG 的不可替代性」：对于需要精确事实锚点的场景（合同条款、实验数据、法规原文），wiki 只能做入口，不能替代原文。V2 自己也承认：「raw 是证据层，源头坏了后面全乱」——这个约束说明 wiki 的适用范围有边界，不能在所有知识管理场景中都取代原始文档的角色。 ^[entities/llm-wiki-architecture]

## 现实案例

本 wiki（Hermes-Wiki）本身就是 LLM Wiki 范式的具体实现：raw/articles/ 作为证据层，concepts/ 和 entities/ 作为编译层，SCHEMA.md 作为规则层，log.md 作为操作记录层。每次收录新资料时，Agent 自动把文章拆成多个节点，用 wikilink 把新节点和已有节点关联，形成一个随资料增多而自动生长的知识网络。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

Obsidian 生态中的多个实现案例：OpenKB 能处理长文档和图片，适合做研究型知识库；Sage-Wiki 用 Go 语言实现，一个二进制文件就能跑；Obsidian 插件版直接在 Obsidian 里使用；Axiom Wiki 是命令行爱好者的选择。这些实现覆盖了从轻量到企业级的不同场景。 ^[entities/karpathy-llm-wiki-v2-2026]

Karpathy 本人的 MenuGen 项目展示了 Software 3.0 思路和 LLM Wiki 思路的结合：直接把菜单照片喂给模型，让模型输出新菜单——中间不需要人工整理知识库，知识管理在 prompt 层面一次性完成。这代表了一种和 wiki 不同的 LLM 原生知识管理路径——不是维护持久化的知识网络，而是让每次查询都自带上下文。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

- [[entities/karpathy-llm-wiki-v2-2026|Karpathy LLM Wiki v2]]
- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki 9 步法]]
- [[concepts/knowledge-network-self-growth|知识网络自生长]]
- [[concepts/wikilink-knowledge-graph|wikilink 知识图谱]]
- [[concepts/karpathy-llm-wiki-v2|Karpathy LLM Wiki v2 概念]]

## 进一步阅读

- [[entities/karpathy-llm-wiki-v2-2026]]
- [[entities/karpathy-llm-wiki-second-brain-awkthole]]
- [[entities/llm-wiki-architecture]]

## 所属 MOC

- [[moc/layer-0-foundation|Layer 0 Foundation]]
