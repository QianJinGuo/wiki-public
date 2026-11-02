---
title: "D2C 的移动端验证闭环：把 74 个 AI 决策点砍到 8 个，再用四层视觉门禁判对错"
created: 2026-11-02
updated: 2026-11-02
type: entity
tags: [d2c, design-to-code, agent-harness, verification-loop, visual-diff, pixel-diff, vision-llm, weex, three-platform, human-in-the-loop, taobao, first-party]
sources:
  - raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02
confidence: 0.9
provenance_state: extracted
review_value: 9
review_confidence: 9
---

# D2C 的移动端验证闭环：把 74 个 AI 决策点砍到 8 个，再用四层视觉门禁判对错

## 一句话结论

AI 生一张 UI 卡片不难，难的是**它不知道自己写错了**——没有眼睛，看不到真机渲染成什么样。淘宝 D2C 团队做的事不是换个更强的模型，而是**给移动端补上 Web 端早就有的验证闭环**：Issue 驱动全自动生码 → 编译 → 三端真机截图 → 像素 diff → 迭代修复 → 提交；同时把"确定性流程"从提示词里搬到 CLI，把 AI 决策点从 **74** 降到 **8**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 框架：质量 = 输入约束 × 生成约束 × 验证闭环

作者先给了一个可复用的拆法（"Agent = Harness + LLM"的具体化）：^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

- **输入约束**：给 AI 看什么上下文（读懂项目代码、拿到设计稿数据、了解业务规则）
- **生成约束**：让 AI 按什么规范写（编码规范、组件复用、目录约定）
- **验证闭环**：AI 写完之后，自己知道对不对、能自己迭代修复

前两项业界方案相对成熟，**第三项在移动端是断的**：Web 端 AI 生完码可以自己跑 Playwright 截图、拿 DevTools 日志、跑 Headless 回归；移动端 AI 写完 DX 卡片或 Weex 组件，看不到真机渲染、判不了对错、更不会自己迭代。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

断成三个具体问题：**①AI 闭眼交卷**（拿不到真机截图、判不了还原度，"看起来 OK"就交付——badge 圆角被截、暗黑模式颜色错、鸿蒙文字截断，AI 自己完全不知道）；**②三端渲染差异靠人肉**（iOS/Android/鸿蒙的字体栅格化、px/dp 换算、暗黑模式行为各不相同，同一份代码三个样子，上线前人工三端各验一遍）；**③端上 UI 技术栈碎片化**（多套渲染框架同时跑，每接一条链路都要从零造一遍验证工具）。作者的定性很清楚：**这些都不是模型能力问题，是 Harness 工程问题。**^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 传统 D2C vs 现在的 D2C

| 维度 | 过去的做法 | 现在的做法 |
|---|---|---|
| 输入要求 | 切好的设计稿 · 图层命名规范 | 给人看的 PRD + 设计稿链接 |
| 验证方式 | H5 模拟预览 | 三端真机截图 + 像素级 diff |
| 还原度判断 | 人肉看"差不多就行" | 硬门槛 · 不达标自动迭代 |
| 修复方式 | 人工修改 → 重新生成 | AI 看 diff 热力图 → 改码 → 重新验证 |
| 交付物 | 一份代码 | 代码 + 三端截图 + diff 报告 + CR |
| 运行方式 | 人坐在 IDE 前操作 | Issue 驱动 · 全自动 Agent |

一次 D2Weex 生码任务共 **9 个阶段**（AST 分析 → 生码 → 编译 → 三端真机预览 → 视觉迭代修复 → 走查闭环），**大部分步骤由 CLI 骨架自动完成，AI 只在少数需要推理的决策点介入**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 核心机制一：让 AI 少做事，结果反而更可靠

这是全篇最有价值的一节，也是作者踩过的坑。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

**做法（失败版）**：把近千行步骤写进一份 SKILL 说明书，让 AI 照着 1→2→3 执行——"步骤写得越细，AI 应该做得越好，对吧？"

**实际结果**：**漏步骤率约 30%**（写了 1→2→3→…→9，实际跑 1→2→4→7→9）；**幻觉式完成**（自信输出"步骤 3 已完成"但从未调用对应工具）；**2 小时白干**，其中 40 分钟卡在"等截图超时"——而这件事一个 `if file.exists()` 就够了。

**根因一句话**：**把确定性流程塞进了概率性 prompt。** 回头看那份 SKILL，大部分内容是流程编排——分支切换、编译重试、URL 管理、截图轮询——**这些本可以固化为程序代码，却被交给了一个"大概率会照做"的概率系统**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

**改造**：把 74 个决策点里的 **66 个确定性步骤下沉到 CLI 命令**，状态持久化到磁盘（上下文压缩、中断、重启都不影响）。分工边界如下表：^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

| AI 做（容错空间大 · 需要推理） | 下沉到 CLI（零容错空间 · 要求 100%） |
|---|---|
| 流程调度：基于 CLI stdout 的 hint 决定下一步（verify / iteration / done / finalize） | 环境检测 · 分支切换 · AST 拉取 |
| 分析设计稿 AST 结构 · 推理布局方案 | 编译 · 重试计数 · 多层门禁 |
| 按 checklist 审查代码 | 预览截图 · diff 打分 · 回滚判定 |
| 看截图判断视觉 bug | 状态持久化 · 报告生成 · CR 提交 |
| 编辑 TSX/CSS 修复 | 通过 stdout hint 引导下一步 · 流程调度不靠 AI 记忆 |

**两层拦截防"漏步骤/假交付"**：①**每步执行前**——CLI 前置校验 state，AI 若跳过 verify 直接跑 done，CLI 立即 `exit 3` 并在 stdout 打"必须先跑 verify"，漏步骤根本执行不下去；②**AI 声称完成时**——**Stop Hook** 最终校验交付契约（pxdiff 达标 / `real_bugs=0` / MR 已建 / CDN URL 等），不满足则反注入指令让 AI 继续跑，假交付根本 stop 不了。作者的原则是：**不寄希望于 AI 自觉，把"AI 会不会漏"变成"CLI 会不会放行"。**^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

**为什么有效——是个数学事实**：假设每步遵守率 95%，74 步全流程正确率 = 0.95^74 ≈ **2%**；缩到 8 步 = 0.95^8 ≈ **66%**。于是**AI 决策点 74→8、漏步骤率 ~30%→<5%**——**"同一个模型，只是让它少做 89% 的事"**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 核心机制二：四层视觉门禁——只有一个 diff 分数不够

**踩坑案例**：一张订单卡片的 badge "3" 圆角上半被白卡背景截切。AI 看到的 diff 只扣 **0.5 分**（被截像素面积极小），心理锚定"分数高应该 OK"，自评"**96% 还原**"，还去修了错的地方（`marginLeft` 微调），**真结构 bug 一路漏诊到最终交付**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

结论：**单靠像素 diff 分不清"渲染抖动"与"真 bug"，单靠 AI 看图会被缩略图和分数误导。** 于是做四层信号融合，每层回答一个不同的问题：^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

| 层 | 回答什么 | 具体做法 |
|---|---|---|
| L1 整图相似度 | **WHERE** — 哪里有像素差异 | 像素 diff · 组件级截图 |
| L2 结构对比（期望 vs 实际） | **WHAT SHOULD BE vs WHAT IS** — 应该长什么样 vs 真机实际长什么样 | **Pixelator AST（期望值）+ weex-devtools DOM 树（真机实际值）逐字段对比** |
| L3 像素采样验证 | **IS IT REALLY** — 是算法瑕疵还是真 bug | 局部像素抽样与颜色距离测算 |
| L4 Vision LLM 语义复核 | **IS IT ACCEPTABLE** — 语义层面是不是真 bug | Vision LLM（Gemini 多模态）判定 `algorithm_artifact` / `real_bug` / `mixed` |

**L2 是关键**：光有分数不够，AI 还得知道实际渲染成什么样才能对症下药——所以不只从设计稿拿期望值，还从真机拿**实际 DOM 树**，双向对比后能定位到具体是哪个字段差了（位置？尺寸？颜色？字号？）。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

**防绕过**：交付门禁 7 项检查中 **6 项有独立交叉验证**——CLI 检测到 badge 被截、AI 却在门禁里填"badge 完整"，必须**提供书面反驳理由**说明 CLI 为何有误，没有反驳直接拒绝交付。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

效果：**视觉检测准确度 60–70% → 91%+；单个 badge 漏诊率 ~50% → ~10%；交付门禁防绕过覆盖 0/7 → 6/7。**^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 核心机制三：3 轮之后必须转人工，走查页把反馈结构化

四层门禁把准确度做到 91%+，**剩下 5–10% 里藏着最难的问题**——而且不是一种形态，是三类完全不同的场景混在一起：机器一视同仁去修，**有的修不动、有的修错方向、有的根本不该修**。实测结论：**AI 自修复在 3 轮内 ROI 最高，3 轮以上必然要转人工。**^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

| 场景 | 表现 | 为什么必须人介入 |
|---|---|---|
| ① 源头问题（设计稿不规范 + 生码折损） | 设计稿有调试残留 / 图层堆叠 / 分组语义错误；生码时 badge 被截、元素丢失、成组方向错 | AI 按错误源结构反复修，分数原地打转；人一眼看出"应该长这样"，指一下，Agent 直接改最终代码绕过源头 |
| ② 算法极限（人眼看不出的假偏差） | 字体像素级差异、5px 内偏移、抗锯齿边缘扣分——人眼几乎无差别，分数已到 91%+ | 改了也拿不到高分，还可能引入新 bug；人判定"这是精度问题不是 bug"，直接放行 |
| ③ 端上必然差异（diff 前提不成立） | 客户端行高 ≥1.2（设计稿可为 1）、三端字体 baseline 不同、mock 数据 vs 真机业务数据、刘海屏安全区 | 设计稿与真机的像素比对前提失效，AI 靠 diff 循环越修越糟；只能靠人的场景判断打破死循环（"这段文字改 1.2 倍行高"） |

**走查页**的设计意图：把"3 轮后转人工"这个动作，从"设计师在钉钉上反馈一些信息再整体给到 AI"的**不标准化流程**，压缩成**浏览器里拖框标注 + 一句问题描述 + 分钟级修复**。三句话概括：**人的输入结构化**（拖框标注位置 + 一句描述 → 结构化写入 Agent 上下文，不是自然语言 bug 单）→ **Agent 上下文精准**（标注已指定"改哪个组件的哪个属性"，直接改，不用重读全组件代码）→ **闭环全自动**（标注 → Agent rerun → 修 TSX/CSS → 回写 → 页面实时刷新，同一页面完成，不切工具）。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 附带约束：生码即分层

生成时就把架构约束写进去：**先分析设计稿 AST 识别可独立单元，做"拆分候选评分"判断哪些该独立成组件，每个组件独立目录、可单独预览与维护，页面组件 = 子组件组合**。理由很实在——一次生码不难，难的是生完之后能持续迭代；如果 AI 生成的是一个大文件（UI/数据/业务混在一起），改一处 UI 要翻一堆逻辑、加一个接口要动 UI 层。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 深度分析

### "让 AI 少做事"是 harness 设计的第一性原理

多数团队遇到 Agent 不可靠的第一反应是"把提示词写得更细"——这篇给出了反例及其数学解释：**每步 95% 遵守率，74 步叠加只剩 2%；8 步则有 66%**。所以真正的问题不是"模型够不够听话"，而是**你让不让它有机会不听话**。把确定性步骤下沉为 CLI、把流程调度靠 stdout hint 而非 AI 记忆，本质是**把可靠性从概率域搬到确定域**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

### 门禁的正确形态是"CLI 有否决权，AI 有举证义务"

"不寄希望于 AI 自觉，把'AI 会不会漏'变成'CLI 会不会放行'"——这句是全文的核心判据。它对应两个具体机制：**前置 state 校验（`exit 3`）**与**Stop Hook 交付契约**；再加上"CLI 说被截、AI 说完整时必须书面反驳"的交叉验证。**这套结构把"模型自我报告"降级为"待证事实"**，比任何"请仔细检查"的提示词都可靠。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

### 单一 diff 分数的失效，是"代理指标"的经典陷阱

badge 案例的教训不止于视觉：**分数只扣 0.5 分是因为被截像素面积极小，而 bug 的严重性不由像素面积决定**。四层门禁的设计恰好拆开了四件事——哪里不同（L1）、应该是什么 vs 实际是什么（L2）、差异是算法瑕疵还是真 bug（L3）、语义上可接受吗（L4）。**每一层都在纠正上一层的代理偏差**，这是"不要用单一指标做验收"最完整的工程示范之一。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

### 人的位置在"打破死循环"，而不是"兜底检查"

三类残留场景有一个共同点：**继续迭代不会收敛**（源头错了 / 到算法极限 / diff 前提失效）。所以人工介入的价值不是"多一双眼睛"，而是**提供机器无法自己生成的先验**（"应该长这样"、"这是精度问题"、"行高改 1.2"）。走查页把这类先验压缩成结构化标注，让 AI 拿到的是**可执行的指令**而不是一段自然语言抱怨——**人的输入质量直接决定 Agent 的收敛速度**。^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

### 三端差异是"验证前提"问题，不是"模型能力"问题

iOS/Android/鸿蒙的字体栅格化、px/dp 换算、暗黑模式行为不同——这意味着**同一份像素 diff 门禁在三端需要各自的基线**，而不是指望模型"理解三端差异"。作者的应对是把差异显性化（三端各自截图与 DOM 对比）并把不可比的部分交给人工判定（场景 ③）。**这条经验对所有"跨环境一致性"类项目都成立：先确认比较前提是否成立，再谈优化。**^[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02.md]

## 实践启示

1. **先补验证闭环，再谈生码能力**：AI 写不对往往不是不会写，而是写完不知道对不对。

2. **按"容错空间"划边界**：需要推理、容错大的交给 AI；要求 100% 的（编译、重试、截图、状态、门禁、提交）下沉到程序。

3. **算一遍 0.95^n**：如果流程有几十步，先把步数砍到个位数，比调提示词有效得多。

4. **状态落盘**：上下文压缩、中断、重启都不影响流程——这是"能重跑"的前提。

5. **让 CLI 拥有否决权**：前置 state 校验 + Stop Hook 交付契约，把"自我报告"变成"待证事实"。

6. **别用单一分数验收**：WHERE / WHAT vs WHAT SHOULD BE / IS IT REAL / IS IT ACCEPTABLE 四问分层，每层纠正上一层偏差。

7. **门禁要能交叉验证**：7 项里 6 项独立交叉验证，AI 与 CLI 冲突时必须书面举证。

8. **明确"几轮转人工"**：3 轮内 ROI 最高；超过就是源头问题、算法极限或前提失效，继续迭代不收敛。

9. **把人的反馈结构化**：拖框 + 一句描述 → 直接定位组件属性，比自然语言 bug 单快一个数量级。

10. **生码即约束**：分层与组件化在生成时确定，否则后续维护成本会吃掉生成收益。

## 与相关实体的关系

- [[entities/design-to-code-loop-figma|Figma 设计稿到代码的闭环]]、[[entities/ai-understanding-component-library-intelligent-d2c-architecture-aws-kiro-mcp-skills|组件库理解与智能 D2C 架构]] 讲**生码侧**（输入约束与生成约束）；本篇补的是**验证侧**——三端真机闭环与四层门禁，三者合起来才是完整的 D2C harness。
- [[entities/taobao-live-search-rec-ai-friendly-architecture-migration-emeya-2026-10-30|淘宝直播搜推 Emeya]] 同属"大淘宝技术 + harness 工程"这条线：都强调**用结构约束替代临场判断**、都要可检查的执行证据（`ExecuteRemark` ↔ diff 报告与门禁契约）。
- [[entities/meituan-distributed-storage-ai-coding-verification-boundaries-2026-10-29|美团 AI Coding 验证边界]] 提供"分层证据链"的方法论，本篇是移动端 UI 的具体落地（截图 + DOM + 像素 + 语义四层）。
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计]]、[[entities/harness-engineering|Harness Engineering]] 讲 harness 的一般结构；本篇给出"74→8 决策点""Stop Hook 反注入""CLI exit 3"这些可抄的实现细节。
- [[entities/agent-data-flywheel-platform-selection-zeus-1688-2026-10-30|Agent 数据飞轮平台选型]] 的"工具调用成功 ≠ 任务完成"与本篇"分数 96% ≠ 视觉达标"是同一类错误的两个现场。

→ [[raw/articles/taobao-d2c-harness-three-platform-visual-verification-2026-11-02|原文存档]]
