---
title: "Agent 的六个自主性等级：从 L0 辅助到 L5 例外管理"
created: 2026-07-22
updated: 2026-09-04
type: entity
tags: ['agent-autonomy', 'l0-l5', 'addy-osmani', 'orchestration', 'agency']
sources: [raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026]
provenance_state: extracted
---

> -> [[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md|原文存档]]

# Agent 的六个自主性等级：从 L0 辅助到 L5 例外管理

Addy Osmani（Google 工程负责人）在《Agentic Autonomy Levels》（2026-07-03）中提出的双轴模型，用 **Agency**（单个 Agent 的行动范围）与 **Orchestration**（多 Agent 的组织方式）两根轴，把 AGI 时代的自主性从"人坐驾驶位"一路拆到"人只管例外"。这六个等级不是给 Agent 贴的荣誉徽章，而是一把衡量"我们把多少控制权、多大失败代价、多少验证责任交给机器的尺子。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

## 摘要

本文是对 Addy Osmani 的 L0–L5 自主性分类体系的中文重述与分析。它把 Agent 的进化分成三个阶段——**Assisted**（人驾驶，Agent 等待指令）、**Agent-led**（Agent 接管有边界任务，人纠正与验证）、**Orchestration**（系统持续分派，人退到处理例外）——并在其上铺开六个可操作的等级。每一级都同时回答三个问题：Agent 接管了什么、人类保留了什么、用什么方式验证。这套框架的价值恰恰在于它把"自主性"从一个模糊的形容词，变成了可用于工程审计的连续刻度。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

## 核心要点

- **双轴而非单轴**: 自主性 = Agency（单 Agent 行动范围）× Orchestration（多 Agent 组织方式），只看任何一根轴都会误判系统所处的位置。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]
- **L0–L5 递进的本质是移交**: 从"建议"到"具体动作""有边界任务""目标驱动""并行委派""例外管理"，Agent 逐步接管执行，人类逐级后退到定义、监督、治理。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]
- **每一级都绑定一种验证方式**: 本地判断、逐次审批、测试/lint/复现步骤、可自动测量的成功标准、独立 workspace 评审队列、独立实现/评审/测试/安全关卡——验证手段必须随自主性同步升级。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]
- **风险与可逆性三问**: 多快能知道做错了？多容易撤销？用什么证据证明做对了？这是决定能不能升一级的判据。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]
- **用指标而非感觉衡量**: 平均人工干预间隔、最长无人值守运行、sandbox 与升级行动比例、批准/拒绝比——这些才是"我们站在哪一级"的客观证据。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]
- **四大反模式是升级路上的暗礁**: Autonomy as status、Permission laundering、Summary substitution、Fleet cosplay，每一条都是把自主性误用当成本事。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

## 深度分析

### 从 L0 到 L1：辅助进入监督的门槛

L0 Assist 阶段，Agent 只给建议，判断与最终执行仍握在人手里，验证靠人的本地判断——这是最低风险的起步位。升到 L1 Supervised Action，Agent 才被允许执行具体动作，但关键操作仍需逐次审批，验证边界落在权限边界与逐次确认上。这一跃的本质是**把"告诉 Agent 怎么做"变成"允许 Agent 动手做"**，代价是人类必须为一个又一个动作点确认键。对编排而言，L1 意味着每个高权限动作都应有明确的人为审批闸门，绝不能因为"我已经授权过了"就变成常开通道。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

### L2–L3：从有边界任务到目标驱动的连续循环

L2 Scoped Task Delegation 是全托管链条的承重墙——Agent 接下一个边界明确的完整任务（如"修这个 bug"），人有明确定义的范围并做监督，验证靠测试、lint、截图与可复现步骤。这一步最难的部分不是 Agent 能不能干活，而是人能否**把任务结界定得足够窄**：边界越清晰，越敢放手。L3 Goal-Driven Autonomy 则更进一步，Agent 围绕目标持续循环，人退到只给目标、规则与停止条件，验证交给可自动测量的成功标准。三级的跨越是编排范式的切换：从"下一单做一单"转向"给定目标自证完成"。这也是[[concepts/agentic-engineering-paradigm]]里"把人从循环里摘出去但保留目标仲裁权"的实践落点。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

### L4–L5：并行委派与例外管理的编排终局

L4 Parallel Delegation 把自主性从单 Agent 推到多 Agent：多个隔离任务并行，人类负责分解、所有权与合并，每个任务跑在独立 workspace 并挂独立评审队列。这是[[concepts/multi-agent-collaboration-patterns]]与[[concepts/agent-orchestration-patterns]]真正上台的地方——没有编排层，L4 的"并行"只会退化成各自为战。L5 Managed-by-Exception 是终局：Agent 持续接收、分派、监控、重试，人只保留 policy、治理与例外决策。此时验证体系必须完备到独立实现/评审/测试/安全关卡都自成闭环，因为人已经不再逐单把关。Addy Osmani 的 Google 背景决定了这套框架带着真实生产负载的痕迹——它关心的不是模型潜能而是**长期无人值守下的可靠性**，与[[entities/agent-intent-recognition-full-evolution]]那种从意图识别端切入的路径互补，一个管"让它自己跑"，一个管"让它别跑偏"。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

### 自主性的伦理与算法两面

这套分级隐含一个尖锐提醒：**越往上，人类的"操控感"越依赖早期失速的恐慌感被验证体系抵消**。L5 不是"做得更好"，而是"设计了足够好的护栏后可以把人撤走"。与之互为镜像是[[concepts/the-agency-model-dangers]]所指出的 agency 放大的风险——自主性越高，模型自身意图漂移与工具误用的代价就被放大得越狠。因此每一级升级都必须回到风险三问（错得多快可知、多容易撤销、证据多硬），用它当作进级的闸门而非勇气测试。^[raw/articles/agent-autonomy-levels-l0-l5-addy-osmani-2026.md]

## 实践启示

1. **先定验证手段，再谈授权等级**。每升一级前，先回答"用什么证据证明它做对了"，证据没设计好就不要往上走。
2. **把 Agent Contract 当一次性配置下发**。每个任务开始前明确 Goal/Scope/Non-goals/Tools/停止条件/Escalation/Budget，契约完备是进级的硬前置。
3. **用指标校准你真实所处的等级**。记录平均人工干预间隔、最长无人值守运行、sandbox vs 升级行动比例、批准/拒绝比，让"我们在哪一级"有数字而非感觉。
4. **警惕 Permission laundering 反模式**。厌倦审批时大幅放宽权限是最危险的升级方式；宁可提升验证自动化也不要代之以权限放开。
5. **L4 及以上必须先有编排层与隔离**。并行委派要配独立 workspace、评审队列与合并策略，否则多 Agent 只是多份乱象。
6. **人封存的不是"做事"而是"例外"**。越高等级，越要把人力的稀缺注意力留给 policy、治理与真正的异常，同时让不可自动验证的操作保留逐次确认闸门。

## 相关实体

- [[concepts/agentic-engineering-paradigm]]
- [[concepts/agent-orchestration-patterns]]
- [[concepts/multi-agent-collaboration-patterns]]
- [[concepts/the-agency-model-dangers]]
- [[entities/agent-intent-recognition-full-evolution]]