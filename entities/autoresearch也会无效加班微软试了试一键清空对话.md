---
title: "Autoresearch也会无效加班？微软试了试「一键清空对话」"
created: 2026-10-15
updated: 2026-10-15
type: entity
tags: [agent, harness, context-management, autoresearch, microsoft]
sources: [raw/articles/autoresearch也会无效加班微软试了试一键清空对话]
confidence: 0.75
---

# Autoresearch也会无效加班？微软试了试「一键清空对话」

## 核心论点：长程 Agent 会困进「思路盆地」

微软研究院对长程 autoresearch Agent（一次任务连续跑几十小时~数天）的分析发现：跑得越久未必想得越多。同一算法优化题、相同配置从同一起点独立跑 8 次，最好 100 分、最差 68.3 分——落后者仍在改代码、提分数，却始终没追上。研究团队把这个过程类比为**梯度下降（Idea Descent）**：分数可微改进，但解法空间可能早已局部收敛（论文：Fork-and-Flush: Escaping Idea Basins in Autoresearch Agents, arXiv:2610.07447）^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。

## 解法空间可视化：3 万次 LLM 比较定义 Idea Basins

仅看分数无法判断 Agent 是否真正换过解法。团队对提交程序去重、抽样后让 LLM 生成统一功能规格说明（保留算法/数据结构/关键参数，滤掉代码风格），再做 3 万余次 A/B/C 相似度判断，把方案映射到二维空间——不同运行的提交聚类在少数**Idea Basins**区域附近，后续提交停留在各自区域。二维映射与留验 LLM 相似度判断平均一致率 **85%**（随机基线 50%），换嵌入方法聚类现象仍在 ^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。

## Fork-and-Flush 机制

运行到预定节点：**复制当前工作空间 → 启动多个新分支**（继承代码、实验文件、历史最佳结果）→ **清空对话上下文** → 各分支独立探索 → 择优继续 → 下个节点重复。要点：

- 分叉时机按**固定预算节点**（开局 4 分支择优，预算 1/3、2/3 处再分叉）——按"成绩停滞"触发的 25 次早期测试中 15 次从未触发：小幅提分会不断重置停滞计时，判断"卡住"不可靠 ^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。
- 调度由 Agent Harness 完成，**无需修改模型权重**；提示词直接提醒"换个思路"在本组实验中没有显著效果 ^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。
- 与"全部预算给一个 Agent"（单盆地内打磨）和"4 个独立 Agent 从零跑"（每条轨迹预算 1/4）相比，Fork-and-Flush 兼顾两者：不丢已验证成果，又强制多方向探索 ^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。

## 实测

13 项长程任务。Polyomino 任务中一次中途分叉出现明显分化：4 分支继承相同工作空间、全部清空历史对话，只有 1 个分支在首次提交就跳入新解法区域（跳转距离达单次运行相邻提交位移中位数的 **43 倍**），最终 93.9 分 vs 单次运行基线中位 84.6 分。作者强调：清空上下文不保证换思路，价值在于让多分支各自尝试、把后续预算交给更有希望的方向 ^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。

## Wiki 视角

这是 harness 层反"上下文惯性"的结构化方案：与 context 压缩（对话历史定期压缩仍无法逃离早期算法选择）互补——压缩保留连续性，Fork-and-Flush 主动制造断裂。对探索机制设计（何时分叉、预算分配、分支选择）是可复用的实证参照 ^[raw/articles/autoresearch也会无效加班微软试了试一键清空对话.md]。

## 关联

- [[entities/agent-harness-context-management-working-set|Agent Harness 上下文管理（Working Set）]] — 同属 harness 层上下文治理，工作集方法管"带什么"，Fork-and-Flush 管"何时丢"
- [[concepts/multi-agent-context-isolation|Multi-Agent 上下文隔离]] — 分支清空上下文即隔离的主动形式
- [[concepts/ai-agent-exploration-path|AI Agent 探索路径]] — 探索/利用权衡的实证案例
- [[concepts/context-engineering|Context Engineering]] — 上位方法论

→ [[raw/articles/autoresearch也会无效加班微软试了试一键清空对话|原文存档]]
