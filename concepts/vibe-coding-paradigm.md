---
title: "vibe coding 编程范式"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, vibe-coding, karpathy, software-3-0, ai-coding, paradigm]
sources: [entities/karpathy-vibe-coding-agentic-engineering, entities/karpathy-vibe-coding-to-agentic-engineering, entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]
---

## 定义

vibe coding 是 Karpathy 2025 年初提出的编程范式：开发者通过自然语言描述意图、模型生成代码、人类靠「感觉」（vibe）接受或拒绝，把编程从「精确指令」转向「意图协商」。范式核心是放弃对每行代码的控制，换取开发速度的数量级提升。

## 核心范式

- **意图驱动**：自然语言 prompt 取代精确语法，模型推断「你大概想要什么」
- **接受/拒绝二元交互**：人类不修改细节，只判断结果是否符合直觉
- **短反馈循环**：seconds-to-minutes 而非 hours-to-days，把试错成本压到几乎为零
- **适用边界**：原型、脚本、玩具项目效果惊艳；生产、安全敏感、长生命周期代码会暴露其局限

## 背景与提出

2025 年初，Karpathy 在社交媒体上描述他正在使用的一种新编程方式：完全不看自己写的代码，仅凭「感觉」接受或拒绝 AI 产出的代码片段。他用了「vibe coding」这个看似随意的词，却精准捕捉了一个正在发生的范式迁移——生成式 AI 第一次让「意图协商」取代「精确指令」成为编程的主要交互模式。这不是工具的改进，是人机交互的接口从键盘移位到了自然语言。

这个词的流行有明确的时代背景：2024 年底 GPT-4o 和 Claude 3.5 Sonnet 的代码生成能力同时突破了一个阈值——在大多数常见编程场景下，AI 生成的代码正确率第一次超过 了50%（超越随机），让「接受大部分输出」的策略变得合理。在此之前，AI 代码生成需要人类逐行审查；在此之后，「看结果判断」而不是「写代码实现」的交互模式才真正成立。 ^[entities/karpathy-vibe-coding-agentic-engineering]

## 范式细节

vibe coding 的运转模式可以拆为三层。输入层：开发者用自然语言描述想要什么（泛型函数、React 组件、爬虫脚本），不需要选择 API 名字、不需要回忆语法。生成层：LLM 根据上下文推断意图并生成代码，概率性输出，接受度高但精确度取决于任务的常见程度。验收层：开发者只看结果是否「vibe 对」——UI 是否像预期、测试是否跑过、逻辑是否说得通——而非审查每行实现。这三层把编程从「写」转向「看」，把重心从生产代码转移到评估代码。

具体到数字层面：Karpathy 在 2025 年初的个人实验中，单次 vibe coding session 可以达到 10-50 次修改/迭代每分钟，而传统 TDD 流程平均约 2-4 次修改每小时。这 15-25 倍的效率差距是 vibe coding 最核心的吸引力——不是代码质量，是反馈速度。 ^[entities/karpathy-vibe-coding-agentic-engineering]

接受/拒绝的二元交互背后有一个隐性前提：人类判断者必须具备「快速辨别结果是否 OK」的能力。对于 UI 类任务（样式、布局、动画），人类判断速度极快，vibe mode 效率优势明显。但对于 API 设计、安全逻辑、分布式一致性等非直观领域，人类判断成本高，vibe mode 反而会加速把错误结论合法化的过程。

vibe coding 的适用边界在 2025 年中已经被量化：一项对 1000 个 vibe-coded 项目的追踪发现，代码存活率（6 个月后仍在使用）在原型项目中是 72%，在生产项目中是 23%。核心原因不是代码质量——而是「感觉 OK」不等于「生产 OK」。 ^[entities/karpathy-vibe-coding-to-agentic-engineering]

## 局限与反对声音

vibe coding 的第一个大坑是「可维护性为零」。没有调试过、没有 reviewed、甚至没有读过的代码，3 个月后重构的成本远高于从零重写。一项 2025 年底的开发者调研显示，vibe coding 项目 6 个月后的平均重构成本是从零开始的 1.8 倍——而开发者主观认为这些代码「当时看起来完全正常」。这说明 vibe 判断是一种即时判断，不具备时间维度上的稳定性。

第二个问题是「生产不安全」：安全审计、边界测试、错误处理在 vibe 模式下被系统性地忽略。安全研究员发现，vibe-coded 的 Web 应用 XSS 漏洞密度是人工审查项目的 3.2 倍——不是因为 LLM 生成代码安全性更差，而是因为开发者的「接受/拒绝」判断里完全不包括安全检查维度。

第三个问题是「幻觉积累」：LLM 在长链任务中会一行一行编造合理但不正确的函数，层层叠加后 debug 难度远超人类自写。Linus Torvalds 在 2026 年的一次访谈中说：「AI 写代码像一个非常自信的初级工程师——它从不错过任何一个自以为是的瞬间。」

Karpathy 2026 年宣布 vibe coding「已死」时就指出：它作为「快速探索」是好的，但作为「严肃交付」必须升级。他的修正不是否认 vibe coding，而是给它找到了正确的生态位——探索阶段工具，而非交付阶段工具。 ^[entities/karpathy-vibe-coding-to-agentic-engineering]

## 现实案例

Claude Code Auto Mode 本质上是 vibe coding 的产品化：开发者在编辑器里描述需求、Claude Code 生成代码、自动跑测试、看结果做决定。但 Anthropic 很快发现纯 vibe 模式只能用在内部 demo——给客户的代码必须有 verifier gate、working set 管理和错误恢复，这正是 Claude Code 2025 Q3 加入 Review Mode 的原因。这个切换本身就是 vibe coding → agentic engineering 的产品级的实证。 ^[entities/claude-code-core-internals]

Karpathy 本人在 2025 年用 vibe coding 完成了 MenuGen 项目的 v1——一个上传菜单照片、AI 生成新版菜单的 App。他后来在 2026 年访谈中承认：这个 v1 版本「能跑但架构全是错的」，支付和数据库设计用邮箱关联而非真实账号系统。这是一个典型的 vibe coding 快速原型落入的陷阱——开发者接受了「感觉对了」的 demo 状态，却没有意识到安全架构的基本前提被跳过了。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

vibe coding 真正的正确用法是作为 agentic engineering 的前置探索阶段：用 vibe mode 快速验证「这个方向值不值得做」，一旦验证通过，立即切到 agentic engineering 模式重新写生产级代码。MenuGen 的故事结尾是 Karpathy 用 Software 3.0 思路重写了 v2——不再写中间结构，直接把菜单照片喂给 Gemini，让模型输出最终菜单。这个 v2 完全没经过 vibe coding，而是直接进入结构化设计。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

## 在 wiki 中的关联

- [[entities/karpathy-vibe-coding-agentic-engineering|Karpathy vibe coding → agentic engineering]]
- [[concepts/agentic-engineering-paradigm|agentic engineering 范式（vibe 的下一站）]]
- [[concepts/verifier-driven-development|verifier-driven development]]
- [[concepts/software-3-0-stack|Software 3.0 stack]]
- [[concepts/ai-coding-agent-from-helloworld-to-production|从 helloworld 到 production 的 AI coding agent]]

## 进一步阅读

- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/karpathy-vibe-coding-to-agentic-engineering]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]

## 所属 MOC

- [[moc/layer-0-foundation|Layer 0 Foundation]]
