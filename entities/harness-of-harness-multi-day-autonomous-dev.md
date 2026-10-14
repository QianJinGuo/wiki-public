---
title: "Harness-of-Harness：多日自主软件开发的编排层框架"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [agent, harness, harness-engineering, coding-agent, autonomous]
sources: [raw/articles/harness-of-harness-paperweekly-firsthand-2026-10-12, raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04]
confidence: 0.8
provenance_state: extracted
---

# Harness-of-Harness：多日自主软件开发的编排层框架

## 框架定位

Harness-of-Harness（HoH）是上海人工智能实验室提出的自主软件开发框架：**不更改 harness 底座**，而是在其之上把整个开发过程组织成反复执行的「规划-编码-测试」循环^[raw/articles/harness-of-harness-paperweekly-firsthand-2026-10-12.md]。此前模智空间的解读将其概括为「编程智能体之上的编排与质量保障层」——规划→开发→测试闭环，让任意（harness+模型）组合都能小步增量、独立验收、证据留痕地持续改进^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md]。两份来源互补：作者一手稿给出动机与基准数字，解读稿给出层级定位视角。

## 动机：AI 编程的方向盘仍在人手里

论文开篇讨论一个基础问题：现有 coding agent 是否已具备「自主开发软件」的能力。就当前主流实践而言答案是否定的——随着大模型能力提升，agent 能完成相当规模的编程任务，但目标设定、验收判断和过程干预仍依赖人，长时程任务会因错误累积与缺乏持续质量保障而失败^[raw/articles/harness-of-harness-paperweekly-firsthand-2026-10-12.md]。HoH 的思路是把「验收与改进」本身工程化：不信任单次生成，而以循环式的独立测试验收驱动多日迭代^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md]。

## 实证结果

- 三个基准上，HoH 相对各自的裸 harness **平均提升 52.25%**^[raw/articles/harness-of-harness-paperweekly-firsthand-2026-10-12.md]
- 案例研究：系统历时 **5 天、70 余轮迭代**，无人干预自主完成一款**可游玩的 FPS 游戏**（只给需求、不碰代码）^[raw/articles/harness-of-harness-paperweekly-firsthand-2026-10-12.md]

## 资源

- 论文：arXiv 2609.01481（Harness-of-Harness: Multi-Day Autonomous Software Development with Continual Improvement）
- 项目主页：flesymeb.github.io/HarnessOfHarness
- 代码：github.com/Flesymeb/HarnessOfHarness

## 深度分析

### 三类失效模式如何映射到循环设计

模智空间的解读把长时程自主开发的困难归为三点：遗忘（早先需求、设计决策、踩过的坑被丢到脑后，局部合理的修改悄悄破坏别处功能）、停滞（失败尝试与次优决策累积，新测试证据又推翻旧假设，陷入反复「检查—修复—再检查」）、自欺欺人（功能缺陷不一定在测试暴露，AI 可能在残缺系统上过早宣布完工）^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:15-15]。HoH 的机制设计恰好逐点对应：计划/报告/历史持久化到文件系统并以渐进式披露按需取详，压制遗忘^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:21-21]；把开发切成小的可验证增量、规划者每轮依据上轮测试报告重新决策，打断停滞^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:19-19]；开发与独立验收权限分离，且 QA 评估的是被冻结的只读快照，杜绝「边改边自评」式的自欺欺人^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:26-30]。

机制层面，这些对应关系被固化为五条硬原则：①在修复与增长间找平衡，保证每轮真实进步；②开发切成范围有限、问题易定位的小增量；③开发中的测试与独立验收分开；④约束可验证产出而非工作流程（框架规定每角色交什么、不许越权，具体推理交给模型与工具）；⑤鼓励复用不重造^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:21-21]。原则④尤其值得注意：框架只在产出物层面设卡、不规定推理路径，这使同一编排层能套在任意 harness 之上而不与其内部机制冲突。

### 编排层在 harness 优化谱系中的位置

近年把 harness 当作优化对象的工作都在底座内部做文章：AutoHarness 用环境反馈合成 harness，Meta-Harness 搜索调优 harness，Self-Harness 迭代诊断修改自己^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:19-19]。HoH 的差异化在于再抬一层：不改任何现有 harness 的内部实现，像「在 OS 之上装项目管理和测试体系」那样架设编排与质量保障层，本质是把软件工程的「迭代式增量开发」原则 AI 化^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:19-19]。这与 wiki 已收录的同属上海 AI Lab 的 [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness]]、[[entities/autodesign-meta-harness-optimization-2026|AutoDesign Meta-Harness]] 构成互补谱系：底座自身进化与上层编排是两条正交的改进轴，可叠加而非互斥。

### A/E 双状态：证据是跨轮次的一等公民

每轮输出两个状态：A（工件状态——源码/配置/资源/项目元数据，保证开发增量延续而非每轮从零重来）与 E（证据状态——哪些行为已证实、哪些缺证据、哪些失败未解决）；二者缺一不可：只有 A 没 E，下轮只能靠猜重建开发历史；只有 E 没 A，则没有可改之物^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:32-32]。消融实验把三个支柱定量化：不更新开发计划（每轮用第一版）掉 8.13 分，不给规划者反馈测试证据掉 6.28 分，不保留上一轮工件掉 7.85 分且 token 从 8.41M 涨到 11.12M^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:42-42]。角色级与迭代级的版本化项目历史进一步提供回滚能力和历史诊断复用：大改动搞崩系统可回滚到已验证状态，相似失败再现可直接翻前一次诊断证据^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:34-34]。

### 提升来自结构而非预算堆叠

三个基准（GameCraft-Bench、FrontierSWE、ProgramBench）× 三组 harness+模型配置，HoH 平均相对提升 52.25%、最高 82.86%，且提升与原始水平无关：原本垫底的 OpenCode+DeepSeek 组合在动作/模拟类游戏与 FrontierSWE 性能类任务上反超最强的 Codex Vanilla^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:38-40]。预算控制实验排除了「多跑几遍」的另一种解释：HoH 只用 2 遍预算（5.67M tokens）得 64.84 分，超过 Vanilla 连续开发 3 遍（6.33M tokens）的 58.24 分——收益来自规划—测试结构本身^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:42-42]。Fusepoint 案例展示了完整过程形态：70 轮迭代呈「初始搭建→能力扩张→趋于稳定」三阶段，第 70 轮 81 个 issue 关闭 65 个，其中 17 个曾被重新打开的 issue 即是被完整记录的回归^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:44-44]。

## 实践启示

1. **编排层优先于底座改造**：不动 harness 内部实现即可让任意（harness+模型）组合持续受益，弱基座组合经编排后甚至反超强基座裸跑^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:19-19,40-40]。
2. **写权限唯一化、验收独立化**：Developer 是唯一写者，QA 是只读第三方且只评估冻结快照——「开发者自评」在结构上不可信，应从权限设计上排除而非靠提示词约束^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:26-30]。
3. **把证据当状态管理**：A/E 双状态跨轮持久化，配合渐进式披露（先给精简索引、相关时才取详情），既防遗忘又不撑爆上下文^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:21-21,32-32]。
4. **三个支柱缺一不可**：计划随证据更新、证据驱动重规划、保留既有工件，消融中各自掉 8.13/6.28/7.85 分，去掉任何一个都会退化为低效重复劳动^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:42-42]。
5. **回归留痕成任务**：被重开的 issue 不静默丢弃，而是进入下轮规划成为明确修复任务，让「改动破坏已验证行为」的代价可见、可追踪^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:44-44]。
6. **权衡成本与适用域**：HoH@3 单任务动辄数百万到上千万 token，实验集中于游戏与特定工程任务且 70 轮仅单案例——适合高价值长周期项目，不宜无差别套用^[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04.md:46-48]。

## 关联

- → [[concepts/harness-engineering|Harness Engineering]]
- → [[concepts/coding-harness-engineering|编码 Harness 工程]]
- → [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六机制]]（该实体将 HoH 列为自我改进机制之一）
- 原文存档：→ [[raw/articles/harness-of-harness-paperweekly-firsthand-2026-10-12|作者一手稿]] · → [[raw/articles/harness-of-harness-multi-day-autonomous-mozhi-2026-09-04|模智空间解读]]
