---
title: "SemaPLC：验证门控的 PLC 代码生成 Agent Harness"
created: 2026-08-26
updated: 2026-09-27
type: entity
tags: [harness, verification, agent, code-generation, plc, industrial, engineering]
sources: [raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# SemaPLC：验证门控的 PLC 代码生成 Agent Harness

上海交通大学联合 Sema 研究团队推出的 **SemaPLC**：把 AI Agent 与 PLC 工程工具链彻底连接起来，让自然语言需求一路走到可编译、可验证、可仿真的控制程序。论文《SemaPLC: A Project-Grounded, Verification-Gated Agent Harness for PLC Code Generation》已在 arXiv 公开，代码在 GitHub 开源。^[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26.md]

## 核心洞察：从「生成代码」到「工程闭环」

PLC 控制的是产线、设备、阀门、电梯和安全联锁——不是「代码看起来对」就可以交差，必须确认程序能不能编译、运行时是否符合需求、异常场景下会不会出错、现场人员能不能理解控制行为。因此 AI 写 PLC 代码不是终点，**工程闭环才是 PLC 智能化开发的关键**——从「AI 帮人写一段代码」变成「AI 参与 PLC 程序从需求到验证的完整流程」。^[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26.md]

## 架构：Agent 即工程闭环

SemaPLC 的核心设计思路是将 AI Agent 与 PLC 编译、运行、变量读写、行为验证、过程仿真等工程能力解耦封装，以工具套件形式提供。上层可接入不同大模型，中间由 Agent 组织任务闭环，下层由 PLC 工具套件执行真实工程动作。^[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26.md]

- **Web IDE**：浏览器工作台，同一界面完成需求输入、代码查看、梯形图阅读、变量监控、验证结果查看和过程仿真，把分散在多个软件里的工作收拢到连续交互流程
- **PLC 工具套件**：负责编译、部署运行、变量读取与强制、轨迹记录、行为验证和仿真生成，围绕 IEC 61131-3 标准的 ST（结构化文本）建立生成、编译、运行、验证工具链
- **可复用性**：工具既被 Web IDE 调用，也通过命令行和 MCP 服务对外提供，是一套可被其他系统、插件或自动化流程复用的 PLC 智能化基础能力

## 核心能力

- **自然语言生成 PLC 程序**：描述控制需求（如四层电梯控制器），生成 ST 程序供工程师审查、修改、完善；适合电机启停、急停联锁、交通灯、停车场闸机、水箱液位控制、电梯运行控制、顺序动作控制等
- **自动验证程序行为**：不只检查能否编译，还检查运行是否符合需求；场景未通过时给出变量变化与预期结果的差异，并让 AI 根据反馈继续修复
- **浏览器过程仿真**：实时读取 PLC 变量并驱动设备元素变化（电梯移动、门开、指示灯亮、水位上升、阀门开关），让变量变成可见动作，利于开发、演示和培训
- **单界面闭环**：把开发、验证、仿真整合到一个浏览器窗口

## 实验验证：验证门控的收益

SemaPLC 的系统实证评估以 LLM4PLC、AutoPLC、Agents4PLC 三个业界代表性系统为基线，所有方法调用相同模型接口，在 GPT、DeepSeek、Qwen、GLM、MiniMax 等 7 个主流大模型上评测。^[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26.md]

- **函数级基准**：117 个独立 PLC 功能块生成任务，由独立于生成过程的形式化验证工具统一判分
- **项目级基准**：65 个真实工业项目任务，覆盖十类工业装置，生成的控制逻辑必须能在既有项目中编译、部署并正确运行

**结果**：函数级在全部 7 个模型上均排名第一，平均通过率 72.6%，比最强基线 Agents4PLC（63.9%）高 8.8 个百分点；即使搭配表现最弱的模型仍达 67.5%，高于所有基线平均水平。项目级从集成编译、静态行为、动态行为三层指标衡量，三项均值均排名第一。^[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26.md]

## 意义

SemaPLC 是「验证门控 Agent Harness」在工业代码生成领域的落地：用形式化验证与真实部署作为 Agent 生成质量的闭环门控，让 AI 编程从「能生成」走向「可交付」。其设计（工具套件解耦 + MCP 对外暴露 + 验证反馈修复循环）可迁移到其他领域的工程代码生成场景。^[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26.md]

## 深度分析

### 为什么「验证门控」对工业代码生成是刚需

消费级代码生成可以容忍「大概率正确」——编译失败或行为偏差的代价通常只是重跑一次。工业控制代码则完全不同：PLC 程序驱动的是产线节拍、安全联锁和物理设备，一个未被发现的行为偏差可能意味着停机损失甚至安全事故。因此工业代码生成的质量定义不是「代码看起来对」，而是「可编译、行为符合需求、异常场景可预期、现场人员可理解」这四层条件的叠加。SemaPLC 的关键判断是把形式化验证与真实部署做成 Agent 的门控（gating），让每一轮生成结果都必须通过这些客观检查才能向前推进，而不是靠模型自我报告或人工经验抽查。这与 [[concepts/verifier-driven-development|验证器驱动开发]] 的思路一致：验证器独立于生成器，判分结果才可信。

### 从「生成范式」到「闭环范式」的量化证据

原文实验给出了一个极有说服力的对比：各方法的静态行为得分相差不足 10 分，动态行为得分却差距悬殊——SemaPLC 动态行为 52.2 分，约为最强基线的 1.7 倍。这说明「代码文本满足需求断言」与「部署到真实运行时后行为一致」之间有一条巨大的鸿沟，静态检查无法跨越，只有真实运行才能暴露。消融实验进一步量化了闭环的逐层贡献：从仅生成的 23.1 分，随规范检查、编译、运行时验证逐层叠加提升到 54.1 分——**每一层验证都不是装饰，而是实打实的分数来源**。这组数据可以推广为一个假设：任何领域的 AI 代码生成，其上限差距主要不在生成侧，而在验证闭环的完备程度。

### 模型无关性：Harness 承重、模型让位

实验中最具架构含义的发现是闭环的模型无关性：接入验证闭环后所有模型均有提升，模型越弱提升越大（最高 +33.3 个百分点），且模型间表现差距显著收窄；即使搭配表现最弱的模型，SemaPLC 仍达 67.5%，高于所有基线的平均水平。这意味着**工程质量的下限由 Harness 决定，而非由模型决定**。对企业而言，这直接改变了技术选型的逻辑：不必绑定某个特定大模型，可以随成本与合规需求自由更换，而质量由验证门控兜底。这正是 [[concepts/harness-engineering-framework|Harness Engineering]] 的核心命题——系统竞争力从模型层转移到 Harness 层——在工业领域的实证落地。

### 架构启示：工具套件解耦 + 门控内嵌 + 多路复用

SemaPLC 的架构选择提供了三个可迁移的设计原则。其一，**工程能力解耦封装**：编译、部署、变量读写、行为验证、仿真等能力不写死在 IDE 里，而是做成工具套件，上层可换模型、中间由 Agent 组织闭环、下层执行真实工程动作。其二，**门控是流程的内生环节**：验证不是生成完之后的附加检查，而是嵌在生成-修复循环里，失败反馈（变量变化 vs 预期差异）直接驱动下一轮修复——[[concepts/harness-loop-architecture|Harness 循环架构]] 的标准形态。其三，**同一套工具多路复用**：Web IDE、命令行、MCP 服务三种暴露方式共享底层工具套件，使「PLC 智能化基础能力」可以脱离单一产品被其他系统集成——基础设施化而非应用化的定位。

## 实践启示

1. **为 AI 代码生成分层设计验证门控**：参照 SemaPLC 的三层指标（集成编译 / 静态行为 / 动态行为），先问「能不能编译」，再问「文本是否满足断言」，最后问「真实运行时行为是否一致」；只做静态检查会系统性高估质量。
2. **让失败反馈结构化，驱动下一轮修复**：验证失败时不要只返回「不通过」，而是像 SemaPLC 那样给出变量变化与预期结果的差异，让 Agent 能据此定位并修复——闭环的价值取决于反馈的可操作性。
3. **用真实运行时的行为验证替代人工经验判断**：实验证明静态分数相近的方法在动态行为上可差 1.7 倍；把程序部署到真实（或仿真）运行时执行并对照参考行为，是区分方法可靠性的唯一硬标准。
4. **把领域工程能力做成可复用工具套件，而非绑进单一产品**：通过命令行与 [[concepts/model-context-protocol-mcp|MCP]] 对外暴露同一套工具，让 Agent、IDE、自动化流程共享基础能力；换模型、换前端都不必重做下层。
5. **接受「模型可换、Harness 承重」的选型策略**：验证闭环对所有模型都有提升且弱模型受益更大，企业可以按成本与合规自由选型，把工程投资集中在验证门控和工具链上，而非押注某个模型。

## 相关实体与概念

- [[concepts/harness-engineering-framework|Harness Engineering]]
- [[concepts/coding-harness-engineering|Coding Harness Engineering]]
- 代码生成评测
- Harness 门控评测
- [[entities/agent-formal-verification-ai-code|Agent 形式化验证 AI 代码]]
- [[entities/llm-as-a-verifier-a-general-purpose-verification-framework|LLM-as-a-Verifier]]
- [[entities/from-spec-driven-to-environment-verification-driven-ai-coding-wu-zuoyan-2026|从 Spec 驱动到环境验证驱动]]

→ [[raw/articles/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26|原文存档]]
