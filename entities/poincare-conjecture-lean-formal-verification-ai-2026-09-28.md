---
title: "庞加莱猜想 Lean 形式化验证：470 万行证明的 AI 协作案例"
created: 2026-09-28
updated: 2026-11-20
type: entity
tags: [ai, formal-verification, lean, multi-agent, mathematics]
sources: [raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28]
confidence: 0.7
provenance_state: extracted
---

# 庞加莱猜想 Lean 形式化验证：470 万行证明的 AI 协作案例

## 事件

一个四人小团队（带头人为丘成桐弟子、研究 Ricci 流数十年的教授 + 一名应届本科生 + AI 协作）用证明助手 Lean 将 Hamilton 与佩雷尔曼的庞加莱猜想证明从头形式化，总计约 470 万行代码，全部通过 Lean 内核检查——无一处 `sorry`（"以后再证"占位）^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md]。

## 关键数据

- 总代码量约 470 万行，其中约 270 万行是最后两周在 ChatGPT、Claude 等 AI 帮助下完成
- 顺着最终定理做依赖分析：真正用到的代码为 14197 个文件、约 402 万行；最长一条引用链串起 353 个文件
- 佩雷尔曼的毕生工作只占全部代码的约六分之一^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md]

## 对 Agent 工程的意义

传统同行评议要数年逐页审读，此次"说了算的换成了机器（Lean 内核），写证明的主力换成了 AI"——形式化验证器充当了可机械检验的 ground truth，这是 [[concepts/scei-formal-proofs]] 所述"验证器驱动开发"在数学大定理上的最大规模实证。AI 承担的是"领队-调度-临时工"式多智能体分工中的底层证明生成，人类专家负责数学结构判断——与 [[entities/formalizing-fermats-last-theorem-claude-lean-anthropic-2026]]（费马大定理形式化）构成同族案例，共同指向"AI + 形式化验证器"是高可靠推理任务的工程范式^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md]。

## 深度分析

### 依赖膨胀结构：23 行定理背后的 402 万行

仓库里最终定理文件只有 23 行^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:148-150]，但它直接和间接拖动的依赖达 14197 个文件、约 402 万行，最长一条引用链串起 353 个文件^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:38-40]。拆开看比例更耐人寻味：佩雷尔曼三篇论文对应约 66 万行，仅占六分之一；其余六分之五全是论文里"默认读者早已懂"的基础数学——分析学约 109 万行、微分几何约 77 万行^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:42-48]。单个结论的代价同样惊人：论文一句话带过的短时存在性，因 Ricci 流方程坐标依赖、需先经 DeTurck 技巧改造成标准热方程，连带 Sobolev 空间与谱理论从零写起，耗掉 89 万行；而支撑整个手术论证的典范邻域定理一项就要 272 万行，占整条依赖链的三分之二^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:52-66]。这说明形式化的真实成本与论文的简略程度成反比——7 页的第三篇平均每页对应 1.4 万行^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:44]，真正的瓶颈不是核心创新，而是沉淀在数学共同体里、从未被写下来的隐性知识。

### 验证器改变分工而非改变数学

Lean 内核并未替人类发明新数学：路线仍是 Hamilton—佩雷尔曼的原证明。它改变的是"谁说了算"——传统同行评议需数年逐页审读建立信任，如今可机械检验的内核把信任问题压缩成一次编译^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:32-34]。分工随之重排：老将定方向、审数学结构，年轻人带着 AI 逐行落码，对错最终由内核判定；人类数学家的精力从"一步步写证明"转向"判断该证什么、揪出 AI 写错的地方"^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:172-174]。这正是 [[concepts/verifier-driven-development]] 在最严苛场景下的极限测试：验证器越强，生成侧越可以放心加速——这一规模化曲线与 [[entities/leap-agentic-formal-theorem-proving-google-2026]] 的趋势一致。

### AI 管 AI：三层编排与命题完整性红线

团队开源的工具包呈现清晰的三层结构：最上层的"领队"是研究者直接对话的主会话，只守数学路线、不写证明；其下是长期后台运行的"调度"智能体，负责拆活、派活和验收；真正动手写证明、挑错、查资料的是一批干完即退出的临时智能体^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:90-94]。调度上还有一条硬红线被明确写进派活规则：绝不为能证出来而削弱命题，发现命题为假也算成功，给出反例即停下报告^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:98-102]。这条"反例即成功"的约定把诚实性从口头要求变成了流程约束，与 [[concepts/multi-agent-collaboration-patterns]] 中"验收者与生成者分离"的原则互相印证。

### 骨架先行：把 sorry 关在主库之外

冲刺期一个关键决策是换干法：先搭骨架——把整段证明的结构全部立起来，暂时证不了的步骤用 sorry 占住，让接口不匹配的问题提前暴露，再逐个审过、定稿、证出^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:106-110]。这些骨架文件单独存放、不进主库，因此成品里依然没有一个 sorry。结果是计划表估出的 4 个多月被压缩到不到一周^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:104-114]。这一"占位外置"策略同时拿到了两个矛盾目标：迭代期的接口即早期反馈，交付期的零占位可信度。

## 实践启示

1. **用依赖审计替代代码量叙事**：评估任何大型形式化（或代码库）时，顺着最终产物做可达性分析比仓库总量更诚实——本案例真正用到的只有 402/470 万行，最长引用链 353 个文件^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:38-40]。
2. **把"读者默认知识"列为预算项**：估算形式化/复现成本时，按论文的简略程度而非页数估算；本案例中背景数学（分析学、微分几何）是核心贡献的约五倍^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:44-48]，预算应预留同等量级。
3. **把"反例即成功"写进智能体派活规则**：明文禁止为可证性削弱命题，并将"发现命题为假并停下报告"定义为成功产出，用流程而非提示词约束诚实性^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:102]。
4. **采用骨架先行、占位外置的推进方式**：先用 sorry 占位把全链路结构搭通、暴露接口错配，但骨架文件独立存放不进主库，使交付物保持零占位^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:106-110]。
5. **多层编排配文件级权限隔离**：领队-调度-临时工三层分工之下，每条并行任务线只准改自己名下的文件，干完交清单、由验收方复核后才入主库^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:98-100]。
6. **在人机交界处保留人类否决点**：定义选择、命题陈述、以及"Lean 里证出来的正是数学家想证的那件事"的确认，必须由人类拍板——验证器保证语法正确，语义对齐仍靠专家^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md:94]。

## 关联

- [[concepts/scei-formal-proofs]] — 形式化证明作为 AI 推理的可验证锚点
- [[entities/formalizing-fermats-last-theorem-claude-lean-anthropic-2026]] — 同族先例：费马大定理 Lean 形式化
- [[entities/lean-scaling]] — Lean 形式化的规模化规律
- → [[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28|原文存档]]
