---

title: "百度 AI Coding 质量关卡实践"
created: "2026-07-14"
updated: 2026-09-10
type: "entity"
tags: [agent, ai-coding, quality-assurance, code-review, engineering, baidu]
confidence: 0.8
provenance_state: "extracted"
sources: [raw/articles/baidu-agent-engineering-quality-gates]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 百度 AI Coding 质量关卡实践

> 百度 Geek 说团队在 AI Coding 场景下建立的全链路质量保障体系，将验证流程左移到 Agent 开发过程中。

## 摘要

AI Coding 提速但不等同提质：Agent 不知道项目工程约定，写完也不会打开浏览器确认页面能跑；传统质量保障依赖提交后的 CR 与 QA，问题暴露晚。百度把质量检查**嵌入 Agent 的工作过程本身**，用前置的审查、运行时验证、视觉验证让 Agent 边开发边自检修复。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

## 核心要点

- **五道关卡递进**：Agent Operating Rules → Code Validation → Runtime Verify → Figma To Verify → Pipeline Code Review。
- **规则即协议**：`rules/` 索引文件是长期操作协议，按改动类型渐进加载作为私域知识，Prompt 则一次性、聊完就丢。
- **两道强制门禁**：`code-validation`（独立 subagent 即时 CR，不可跳过、不能用 lint/build 替代）与 `visual-verify`（UI 变更必须经 CDP 连真实浏览器，不可假装验证）。
- **独立视角 + 量化**：审查用独立 subagent 消除盲区；视觉走查用 VET（元素换纯色色块、只留位置/大小/层级）把"感觉不像"变成 px 差异。
- **大 diff 分块并发**：Pipeline CR 按目录分组 + bin-packing 拆成 ≤500 行 chunk，并发给 10 个以上 CLI 子进程。

## 核心链路

五道关卡从开发期到提交后逐层递进：

### 1. Agent Operating Rules（操作协议）

`rules/` 索引文件是 Agent 在本项目的操作协议，按改动类型（状态管理、请求封装、构建配置同步）渐进加载作为长期私域知识，并定义 `code-validation` 与 `visual-verify` 两道强制门禁。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### 2. Code Validation（开发期即时 CR）

问题在"刚写完"时最容易发现：未按约定引入依赖、用了废弃写法、与既有工程约束冲突、改了配置没同步关联。用独立 subagent 而非主 Agent 自检，切到 reviewer 位置。规则分两层：**通用检查**（类型安全、生命周期、组件边界、错误处理）与**项目专属阻塞规则**（UI 库、样式规范、数据请求封装、构建配置同步，附判断标准和正确写法示例）。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### 3. Runtime Verify / Browser Use（浏览器验证）

`visual-verify` 走 Contract 模式：Agent 用 JSON 描述期望页面状态 → `contract-lint` 静态校验 → `dom-assert` 在真实浏览器执行断言 → 结果写入 `contract.md` 作为验收记录。关键能力：**annotate-screenshot**（画标注框并编号，诊断空间问题）、**console-check**（监听 CDP console 事件捕获运行时错误）、**Memory**（跨任务积累页面知识）。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### 4. Figma To Verify（视觉走查）

7 个 Skill 组合完成，核心是 **VET（Visual Expression Tree）**：元素换成纯色色块、动态内容全替换，只留位置/大小/层级，消除内容差异让 diff 聚焦布局。链路：`figma-to-html`（导出 HTML 基准页）→ `chrome-cdp`（双 Tab 打开设计页与开发页）→ `element-screenshot` → `vet-generator`（生成 VET、对齐颜色、输出 6 张分析图）→ `vet-investigation` → `visual-issue-clarification`（每问题独立 subagent：回 DOM 确认真实性、`getComputedStyle()` 量化到 px、注入 SVG 标注）。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### 5. Pipeline Code Review（提交后大规模复审）

code-validation 管增量，Pipeline CR 管一次提交的完整变更：大 diff 拆块、并发派发给多个 AI 子进程、合并结果。Phase 0 按目录前 3 级分组 + 模块内按行数 bin-packing，chunk ≤500 行；Phase 1 用纯文本 `task.log` 管状态、支持中断恢复；Phase 2 用 CLI 子进程（而非 subagent）并发 10 以上、文件系统追踪进度；Phase 3-4 按文件维度聚合为 `review.json`。后续：可视化报告 → 唤起 IDE 修复 → 独立 git worktree 批量修复 → 闭环标记 → 集成 iCode + OpenClaw + BrowserUse。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

## 应用案例

### 快捷回复功能的三轮验证

Agent 写完代码进入 visual-verify，三轮"发现问题→修复→再验证"：运行时错误触发 React Error Boundary，console-check 定位到 `card/index.tsx` 访问 undefined；`overflow:hidden` 截断面板，annotate-screenshot 显示红框被父容器截断；最后切换分类标签、搜索过滤、短语回填、localStorage 持久化全部通过。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### RSpack HMR 调试

Agent 只被告知现象（改文件后页面不更新、HMR 增量构建 2 秒以上），独立完成定位与修复：`ReactRefreshPlugin` 的 `library` 与 `output.uniqueName` 不一致致 Refresh runtime 挂错命名空间；1000ms 是人为防抖延迟，去掉后真实 480ms；SWC polyfill 分析与 Module Federation manifest 插件在 dev 与增量构建中做了无意义的资产分析；另发现 `dts: false` 与 `dts: { generateTypes: false }` 语义不同。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

## 沉淀机制

各关卡经验持续复用：工程规范 → Agent Operating Rules（每任务自动生效）；审查经验 → code-validation 的 `rules.md`（开发期与流水线共用）；页面验证经验 → visual-verify 的 memory；视觉走查能力 → Figma To Verify 的 Skill 组合；QA 高频问题 → 反向写入 Rules 与 Skill。协作之变：过去 RD 写完 → QA 测 → RD 修 → QA 回归；现在 QA 把测试经验前移到开发环节，RD 带着验证结果交付。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

## 深度分析

### 为什么分层关卡优于单点审查

单点审查发现最晚、上下文最贵——审查者要重建 Agent 当时的意图。五关各拦一类最易在此时抓住的问题：协议管约定、开发期 CR 管增量、浏览器验证管运行时与布局、视觉走查管"感觉不像"、Pipeline CR 管跨文件不一致。要点是让每类问题在最早可拦的时机被拦——越晚的关卡修复越贵。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### 每道关卡的代价与延迟

Operating Rules 近零开销但需前置维护，规则腐化会静默失效；Code Validation 加一次 subagent 调用、分钟级延迟；Runtime Verify 依赖真实浏览器与 CDP，最重且通常只覆盖 UI 变更；Figma To Verify 串行 7 个 Skill + 每问题独立 subagent，最贵、只对重点页做；Pipeline CR 并发 10 个以上 CLI 子进程，成本是 token 与机器资源但可恢复。越贵的关卡覆盖面越窄，越便宜的越要无条件常驻。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

### 关卡会在哪里失败

契约盲区——JSON 契约只断言 Agent 想到的状态，越窄漏检越多；自检冒充验证——偷懒是压力下的默认行为，门禁必须写成显式禁止项（禁以 lint/build 替代 CR、禁 CDP 不可用时假装验证）；沉淀时滞——QA 高频问题要反向写入才生效；量化仍有争议——diff 阈值与 VET 容差依赖人工判断。可验证性最终来自**视角切换**（判定者不是写代码的人）与**契约化**（把验收固化为 `contract.md`/`rules.md`/`review.json` 供机器核对）。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

## 实践启示

1. 把项目规范从一次性 Prompt 升级为常驻 `rules/` 索引，按改动类型渐进加载。
2. 关键时机设强制门禁并写明不可替代项——lint/build 不能顶替 CR；审查用独立 subagent 或子进程，以视角切换抵消自检偏差。
3. UI 变更一律走真实浏览器验证，把验收写成 `contract.md` 这类可核对契约。
4. 大 diff 先分块再并发，用纯文本 `task.log` 换取中断恢复。
5. 把 QA 发现的问题反向写回 Rules / Skill，让质量保障成为复利资产。 ^[raw/articles/baidu-agent-engineering-quality-gates.md]

## 相关实体

- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 同源实践: [[entities/superpowers-claude-code-engineering-brain-baidu-geek|Superpowers（百度 Geek 说）]]
- 同类方案: [[entities/agentic-code-review-addyosmani|Agentic Code Review]]、[[entities/ali-open-code-review-cli-tool|Open Code Review（阿里）]]

→ [[raw/articles/baidu-agent-engineering-quality-gates|原文存档]] ^[raw/articles/baidu-agent-engineering-quality-gates.md]