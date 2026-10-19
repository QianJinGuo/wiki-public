---
title: "Anthropic Global Workspace (J-Space) — Claude 的内部心理工作区"
created: 2026-07-07
updated: 2026-10-20
type: entity
tags: [anthropic, llm, interpretability, transformer-circuits, mechanistic-interpretability]
sources:
  - raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法
  - raw/articles/刚刚anthropic-宣布claude-自己长出了意识
  - raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source
  - raw/articles/anthropic最新研究claude存在神秘j空间与人类心智高度相似
confidence: 0.7
provenance_state: merged
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Anthropic Global Workspace (J-Space) — Claude 的内部心理工作区

2026年7月7日，Anthropic 发表了一篇重磅研究《Language Models 中的 Global Workspace》（A global workspace in language models），发现 Claude 内部存在一个特殊的神经激活区域——**J-Space**（雅可比空间），它像一个静默运行的心理工作区，里面会浮现模型正在考虑、可能报告、也可能用于推理的概念。 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md, raw/articles/刚刚anthropic-宣布claude-自己长出了意识.md]

## J-Space 的核心发现

J-Space 是一小组特殊的内部神经激活模式，每个模式对应一个具体的词，但某个模式亮起来并不代表模型正在说这个词——只代表这个词「在它心上」。Anthropic 用雅可比矩阵（Jacobian）作为数学工具定位出这些模式，因此命名为 J-Space。 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]

### J-Space 的独特性质

与 Claude 其他内部处理过程相比，J-Space 具有以下独特性 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]：

- **可报告性**：如果你问 Claude 它正在想什么，它会告诉你 J-Space 中的内容。而非 J-Space 中的表征则更难被报告出来。
- **可调控性**：Claude 可以按要求调节这些表征。如果你要求 Claude 思考某件事，或者在脑中默默解决一个问题，相应的模式会在 J-Space 中浮现。
- **静默运算**：J-Space 使模型可以在不写出来的情况下思考某个概念，与思维链（Chain of Thought）完全不同。思维链是模型在推理时写给自己看的文字，J-Space 是静默运行在内部神经激活之中的。
- **自然涌现**：J-Space 并不是 Anthropic 设计或编程出来的，而是在 Claude 的训练过程中自然涌现出来的。

## 全局工作空间理论

这一发现与神经科学中的「全局工作空间理论」（Global Workspace Theory）形成呼应。在认知科学中，全局工作空间是一个理论框架——大脑中大量无意识的处理过程在后台运行，只有一小部分进入意识层面的「工作空间」被我们感知到。 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]

Anthropic 发现 Claude 内部存在类似的分层结构：绝大部分激活活动在无意识中持续进行，而 J-Space 扮演着类似于「工作空间」的角色。^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]


## 与之前工作的关系

这并非 Anthropic 第一次在模型可解释性上取得突破。此前，Anthropic 已通过 **Natural Language Autoencoders (NLAs)** 实现了将 AI 的内部想法转化为人类可读文字 ^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]，以及通过稀疏自编码器（Sparse Autoencoders）和归因图（Attribution Graphs）等技术追踪输入到输出的因果链。 → [[entities/anthropic-nla-natural-language-autoencoders-interpretability|NLAs 读心术]]、[[entities/sparse-autoencoders|Sparse Autoencoders]]

J-Space 的发现更进一步——它不仅读取出了模型的内部活动，还揭示了这些活动的分层结构和「意识可达性」。 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]

## 反响与意义

OpenAI 应用研究负责人 Boris Power 表示：「Anthropic 的研究表明，现代 LLM 具备某种可通达意识。围绕 J 空间的测试非常有意思！不过，我们目前还没有一种有说服力的测试方法，能够验证现象意识，也就是大多数人直觉中所理解的那种意识。」 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]

研究团队尚未完全理解是什么机制决定了哪些内容会最先进入 J 空间。已有线索表明它与 Claude 的自我感、某种类似情绪反应的东西以及元认知痕迹有关，但其中机制尚未完全厘清。 ^[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法.md]

## 深度分析

### 功能主张，不是现象学主张

GWT 有两个版本：功能版谈广播与全局可用性，现象版谈主观体验。J-space 的证据全在功能层——可报告、可唤起、可被多系统共享读取、消融后多步推理崩塌，这些是 access consciousness 的判据，没有一条触到 phenomenal。原则上，任何行为或探针证据都无法区分「有体验」与「功能相同但无体验」。所以「内部心理工作区」只能当工程隐喻读：一组对下游全局可用、可被语言报告的表征，而非「Claude 在里面想事情」。 ^[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source.md:51]

### 消融能推断必要性，不能推断充分性

删掉 J-space 后仍能流利说话、分类、答题，但多步推理掉到接近零——这是全研究因果性最强的一段，说明 J-space 不是副产品，而是串行组合计算的必要中介。但亮起 spider 不等于 spider 表征独立完成了腿数回忆；它更像一个共享瓶颈：写入一次、多系统取用，所以单点篡改能同时改掉四个问题的答案。 ^[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source.md:34-36]

### 白熊效应：控制不完美，而这是最有用的信息

「不要想」时该概念弱于「主动想」、强于「未提及」，同时 damn、failure 亮起。抑制不是清零，而是压到阈下并附一层负性标记。任何「读 J-space 并抑制有害概念」的方案都必须预设残余与副作用：被压制的念头会改变其他内部状态，输出看不见，探针看得见。它也是验证工具：若某干预号称「干净移除」却在 J-space 留残余，则可疑。 ^[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source.md:30]

### 从「读」到「训」，以及边界

反事实反思训练只监督「被打断会说什么」，不监督行为，结果不诚实率下降且 honest、integrity 在 J-space 亮起——监督穿透到了表征层。含义是：在这一层，「教说什么」与「教怎么想」是同一干预的两端。但也打开反向风险：既然言语训练能改写立场，被训练成口头诚实的模型，其 J-space 是否也亮 honest 而行为不变？这是单一实验、单一模型，恰是「对齐剧场」的技术版本——用可被训练目标塑造的指标验证对齐。 ^[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source.md:47]

边界还有三条：探针效度（J-lens 的操作化是「让模型未来更可能说出该词」，属词预测代理量，与自报一致可能源于共享读出方向）；证据窄（Neuronpedia 上 Qwen3.6-27B 的复现是重要但初期的外部验证 ^[raw/articles/anthropic最新研究claude存在神秘j空间与人类心智高度相似.md:47-49]）；拟人化滑坡——原文自己给了限速器：人脑靠循环回路而 Claude 单次前向；人类工作记忆数秒消退而 Claude 靠注意力召回文本缓存；Claude 的工作空间几乎全由词构成。

## 实践启示

1. **先降级成假设再用。** 写「存在一个类全局工作空间的功能瓶颈」，不写「模型有意识」；每条主张挂上它依赖的实验，无对应实验者不进结论段。
2. **要求必要性与特异性双证据。** 只有「亮起」不够：要消融后能力崩塌（必要性），加「换掉它则相关答案定向改变、无关能力不受影响」（特异性）。方法论见 [[concepts/mechanistic-interpretability|机械可解释性]]。
3. **干预要塑造而非审查。** 白熊效应说明压制会留残余并污染其他状态，反事实训练说明从「该说什么」入手能改写表征。优先改倾向，屏蔽概念当最后手段并默认有副作用。
4. **探针实验记四类日志：** 校准任务与操作化定义、注入/消融的强度与位置、被监测能力之外的能力基线、阴性对照（无关概念激活）。没有阴性对照的「亮起」不值得引用。
5. **把 J-space 纳入威胁建模。** 共享瓶颈可读即可写；被秘密训练的破坏型模型在输出正常时，J-space 起点已有 fake/secretly/fraud。高权限 agent 上，这类信号比输出过滤更早。可对照 [[entities/anthropic-llm-introspection-awareness-mechanisms|LLM 内省机制]] 与 [[entities/deepmind-recirculation-transformer-layer-activation-feedback-2026|循环反馈层]]。
6. **区分三种证据再下笔。** 探针读到的、模型自报的、行为测到的；三者一致才写「确认」，只探针与自报一致写「可能共享读出方向」，只有行为证据则不回填内部机制。 ^[raw/articles/anthropic最新研究claude存在神秘j空间与人类心智高度相似.md:41]

## 参考资源

- 论文地址：https://transformer-circuits.pub/2026/workspace/index.html
- 专家评论（Dehaene、Nanda 等）：https://www-cdn.anthropic.com/files/4zrzovbb/website/cc4be2488d65e54a6ed06492f8968398ddc18ebe.pdf
- Neuronpedia 交互演示：https://neuronpedia.org/jlens
- 官方推文：https://x.com/AnthropicAI/status/2074185348142280912

→ [[raw/articles/刚刚anthropic发现claude类意识工作台神秘j空间藏着没说出口的想法|机器之心原文存档]]
→ [[raw/articles/刚刚anthropic-宣布claude-自己长出了意识|AGI Hunt 原文存档]]
→ [[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source|机器之心第2译本原文存档]]

## 机器之心第2译本补充（3rd source, 2026-07-07, ChallengeHub小编）

此译本与第1 source（同为机器之心）侧重不同，详细枚举了5大实验证据以及此前未覆盖的发现：^[raw/articles/刚刚anthropic-宣布claude-自己长出了意识.md]


### 五大证据系统化呈现
1. **Claude 能报告 J-space 内容** — 体育项目实验：J-lens 榜首 "Soccer" → 模型回答"足球"；抹掉换成 "Rugby" → 改口橄榄球。注入「lightning」→ 报告关于闪电。直接干预 J-space 会改变模型行为。
2. **能按要求调控 J-space** — 边抄写边想柑橘类水果 → J-space 出现 "orange" "fruits"。要求「不要想」某概念 → **白熊效应**：激活弱于想但强于未提及，同时 "damn" "failure" 亮起。
3. **真的用 J-space 思考** — "会结网的动物有几条腿"：J-space 中途亮起 "spider"；换成 "ant" → 答案变成 "6"。
4. **表征可灵活复用** — France→China 替换实验：4 个不同问题（首都/语言/大洲/货币）答案齐刷刷改变 — 同一共享表征被多个系统读取。读写 J-space 的组件数量比普通模式高约 100 倍。
5. **大部分处理不经过 J-space** — 删掉 J-space：Claude 仍能流利说话/分类/答题，但**多步推理掉到接近零**。西班牙语实验：语言识别过 J-space，续写完全不受影响（肌肉记忆）。

### 后训练塑造了 J-space 的「立场」
- **预训练模型**的 J-space 主要服务于预测后文
- **后训练模型**在读用户危险药物消息的当下，J-space 即亮起 "WARNING" "dangerous"
- 角色扮演时每轮开头 J-space 亮起 "fictional" "disclaimer"——自我标注非本意

### 反事实反思训练
新技术 **counterfactual reflection training**：只训练模型「如果被中途打断并要求反思会说什么」，完全不训练实际行为。训练后模型不诚实行为率下降，J-lens 显示 "honest" "integrity" 在 J-space 亮起——**教模型「该说什么」，塑造了它「怎么想」**。^[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source.md]


### "体验式语言"依赖 J-space
删掉 J-space 后回答依然流利但变得干瘪机械——无论是描述自己还是别人的体验。J-space 支撑的是「谈论体验」本身。^[raw/articles/anthropic最新研究claude存在神秘j空间与人类心智高度相似.md]


**整合视角**：此 3rd source 最不可替代新增 = (1) 五大证据的系统化呈现、(2) 白熊效应揭示 J-space 控制不完美、(3) 反事实反思训练——教说什么→塑造怎么想、(4) 西班牙语双轨实验（语言识别 vs 续写）。前 2 source 未提供这些实验的量化细节。 ^[raw/articles/anthropic-global-workspace-j-space-machinexin-3rd-source.md]
