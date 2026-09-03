---
title: "Agent 角色专业化"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, agent, role, specialization, multi-agent, prompt-engineering]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏, entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]
---

## 定义

Agent 角色专业化：把通用 agent 拆成多个专门 agent（researcher / coder / reviewer / writer），每个用极度定制的 prompt + tool set + context。专业化的收益不在「能力」而在「一致性」——通用 agent 跨任务漂移，专业化 agent 跨调用稳定。

## 核心范式

- **prompt 极度场景化**：不写「你是 AI 助手」，写「你是只做 X 的 Y 角色，禁止 Z」
- **tool set 裁剪**：reviewer 不给写文件权限，coder 不给 fetch 互联网
- **输出格式约束**：每个角色规定输出 schema，方便下游 agent 解析
- **风格 / 语气一致性**：同一角色多次调用产出格式高度一致，便于聚合

### 背景与提出

agent 角色专业化的实践源头来自软件工程的「关注点分离」（Separation of Concerns）原则——1980 年代模块化编程就已经知道，把复杂系统拆成职责单一的模块，每个模块更容易开发和维护。但把这条原则应用到 LLM agent，却直到 2023 年下半年才被系统性地总结出来。

原因是早期 agent 的 prompt engineering 走了一条弯路：为了让 agent 更能干，system prompt 越写越长，从「你是 AI 助手」发展到「你需要同时具备 X、Y、Z 能力，且要遵循 A、B、C 规则」，结果是多能力互相干扰，同一个 prompt 在不同任务上表现飘忽不定。角色专业化的出现是对这个问题的反向解答：不是增加 prompt 的复杂度，而是把复杂度分配到多个独立的、专业化的 agent 实例里，每个实例只做一件事但做到极致。

[[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw]] 框架是这个方向最重要的推动者。它的核心主张是：一个「全知全能」的 agent 是不可能稳定的，真正 production-ready 的多智能体系统，应该是每个角色（planner/coder/reviewer/tester）都是高度专业化、极度稳定的 agent，通过标准化接口协作 ^[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]。

### 范式细节

**prompt 极度场景化**是角色专业化的第一原则。「你是只做 X 的 Y 角色，禁止 Z」比「你是一个有帮助的 AI 助手」要好太多，因为后者在不同调用时会产生不一致的行为——同一个人格设定在不同任务上，模型的解读可能存在细微差异。场景化的 prompt 通常包含：角色身份定义（who）、任务边界（what to do / what not to do）、输出格式（output schema）、以及质量标准（what "good" looks like）。一个反面案例是 coder prompt 里包含「同时要检查代码可读性」——检查代码是 reviewer 的职责，coder 的 prompt 里出现这个指令会导致 coder 的输出在「写代码」和「评价代码」之间摇摆。

**tool set 裁剪**是实现角色隔离的物理手段。Coder 不给写文件权限，reviewer 不给 fetch 互联网——这些限制不只是安全考量，更是行为约束。给 reviewer 提供 web fetch tool，它可能会自己去验证代码逻辑，而不是专注于读代码找 bug；给 coder 提供 read tool，它可能会去读取大量无关文件而不是直接写代码。Tool set 的裁剪原则是：每个 role 只拥有完成其核心职责所必须的工具，多余的工具只会增加它「做错事」的可能性。

**输出格式约束**在多 agent 系统里尤为重要，因为下游 agent 需要解析上游 agent 的输出。JSON schema 是最严格的格式——规定每个字段的类型、枚举值、是否必填，LLM 输出不符合 schema 时可以自动重试。Markdown schema 更灵活但解析更复杂。实践中，许多团队发现规定「每段话不超过 3 句」「每行不超过 80 字符」这样的细粒度格式要求，比单纯规定 JSON schema 更有助于让 LLM 输出稳定一致。

**风格 / 语气一致性**是一个被低估的收益。当 coder 被调用 100 次时，它的输出格式应该高度统一——每次都生成同样结构的代码块、同样格式的 commit message、同样风格的注释。这种一致性使得下游的 aggregator（通常是 orchestrator）不需要处理五花八门的格式，降低了聚合逻辑的复杂度。更进一步，当某个角色出现问题需要调试时，高一致性的输出让 diff-based debugging 成为可能——只需要对比两次调用的输出差异，就能定位 prompt 变化带来的行为漂移。

### 局限与反对声音

角色专业化的最大问题是「角色边界的主观性」。谁来定义角色 A 和角色 B 的边界？如果边界定义不清，两个角色之间就会出现「都认为自己不该负责」的空白地带。实践中这个问题的解法是「先运行，再调整」——先用粗糙的角色定义跑通整个 pipeline，然后识别哪些问题是因为角色边界不清导致的，再微调 prompt。[[concepts/multi-agent-team-coordination|多智能体团队协作]]中，orchestrator 的一个核心职责就是处理角色边界不清导致的争议。

第二个批评是「角色数量膨胀」。当系统需要处理 50 种不同类型的任务时，是否需要 50 种不同的角色？角色数量过多会导致：每个角色的训练数据变少（prompt 难以泛化）、orchestrator 的调度逻辑变复杂（需要更精细的任务分类能力）、运维成本上升（50 个 agent 实例的监控 vs 5 个的监控完全不是一个量级）。实践中，角色数量通常控制在 3-7 个，用「能力域」而非「任务类型」来划分角色。

第三个问题是「角色间的知识共享障碍」。当 coder 需要用到 reviewer 对上一版代码的分析结果时，这个信息传递依赖 orchestrator 的聚合逻辑。如果 orchestrator 没有正确地把 reviewer 的发现传达给 coder 的下一次调用，coder 就会重复同样的错误。知识共享的另一个代价是 context 需要包含更多历史信息，而信息过载正是 [[concepts/working-set-vs-long-term-memory|working set vs long-term memory]] 中讨论过的问题。

### 现实案例

[[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw]] 框架在 [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2|OpenClaw 团队搭建经验]] 帖子里记录了一个实际案例：团队设计了 4 个角色（researcher、coder、reviewer、qa），researcher 负责用 @web 工具收集技术文档并摘要，coder 负责根据摘要写实现，reviewer 负责对照 researcher 的文档检查代码质量，qa 负责根据 coder 的实现补充测试用例。上线第一个月，4 个角色的平均任务成功率分别是 89%、76%、68%、82%——coder 成功率最低，因为 reviewer 发现的问题频繁需要打回重写。团队后来调整了角色边界：reviewer 改为和 coder 并行运行而非串行（reviewer 看上一版，coder 写新版，同时进行），整体成功率提升到了 85%。

prompt engineering 模式 中的「角色扮演」技巧就是角色专业化的一个简化版本：把 system prompt 写成「你是一个 X 领域的资深专家」而非「你是一个有帮助的 AI」。这种技巧在单 agent 场景下就能显著提升输出质量，证明了角色专业化思想的普适性。

## 在 wiki 中的关联

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw 角色设计]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2|OpenClaw 团队搭建经验]]
- [[concepts/multi-agent-team-coordination|多智能体团队协作]]
- prompt engineering 模式
- [[concepts/ai-agent-patterns|AI agent 模式]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
