---
title: "ageval：浙大 ZJU-REAL 开源的插件化解耦 Agent 评测底座"
created: 2026-09-12
updated: 2026-11-18
type: entity
tags: [agent, evaluation, harness, agent-eval, plugin, sandbox, open-source, coding-agent]
sources: [raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026]
confidence: 0.7
reviewed: 2026-10-26
review_verdict: keep
---

# ageval：浙大 ZJU-REAL 开源的插件化解耦 Agent 评测底座

> **Background**：本文基于量子位对浙江大学 ZJU-REAL 团队开源项目 ageval 的介绍文章综合整理，聚焦其「评测脚手架与运行时解耦」的架构设计。项目主页 <https://zju-real.github.io/ageval>，仓库 <https://github.com/ZJU-REAL/ageval>。

## 要解决的问题：换一个运行时，评测结果就不可比

ageval 的出发点是 Agent 评测里一个被长期忽视的变量：**同一套模型，换一个 coding agent 运行时，测试得分与实际消耗往往大相径庭**；而在本地、Docker 与各类云沙箱之间来回适配，又会陷入写不完的「胶水代码」。结果是公开榜单只公布模型名与分数，既不说明用了哪套 Harness，也不标注沙箱环境版本与 Prompt 模板，外界难以复现。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

ageval 的核心思路是把**待测 Agent 与运行环境通过插件彻底解耦**：在配置文件中修改一行（`profiles.yaml`），同一份 dataset 就能在不同环境、不同 Agent 之间自由切换运行，无需改动 dataset 本身。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

## 架构一：一次运行的五个标准阶段

ageval 的执行底座是一条**确定性的单向流水线**：`lock → environment → run → evaluate → record`。无论运行过程是成功、失败、超时还是被外部中断，cleanup 钩子都会在 finally 阶段可靠执行，彻底清理容器实例、注销网络并擦除临时凭证。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

三个阶段承担了「评测可信性」的关键职责：

- **lock（静态前置校验）**：在拉起运行环境之前先静态解析依赖图（ExtensionGraph），比对环境是否满足 Agent 插件声明的能力（capabilities）要求，并检查宿主 Docker 守护进程与 API Key 凭证。校验不通过直接在 lock 阶段报错终止，避免跑到一半才因环境缺失崩溃。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]
- **evaluate（独立评分 + 参考答案隔离）**：评分逻辑统一收敛在 `evaluator.py`，支持程序化断言、产物 diff 校验或 LLM-as-a-judge；参考答案（gold）放在 `tasks/*/evaluation/` 目录，在 Agent 执行阶段对环境完全不可见，直到 evaluate 阶段才挂载上传至打分环境，打分容器还可配置 `network: none` 断开外网，彻底防止泄题或作弊。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]
- **record（不可变证据链）**：运行结束后，执行配置与拓扑快照 `lock.json`、评分细节 `result.json`、完整交互事件轨迹 `trajectory.jsonl` 一体化封存归档，保证每次评测有据可查、可精确复现。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

排除参考答案泄漏与网络外联，是 [[concepts/evaluation-harness-design|评测 Harness 设计]] 中「评分环境隔离」一类问题的具体工程解法；五阶段流水线则把「环境准备 / 任务循环 / 评分」的耗时与失败点显式暴露出来。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

## 架构二：基于 export / inject 服务契约的插件机制

框架内部没有面向特定环境或特定 Agent 的硬编码 if/else 分支，所有组件都通过**声明式的服务契约**解耦：插件对外 export 服务、并声明自己依赖的环境 capabilities，由框架 inject。插件详情页会直观展示该插件对外 export 的服务、依赖的 capabilities 以及安装命令与参数配置示例。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

插件分两类，可自由组合：

| 插件类别 | 作用 | 已收录示例 |
|---|---|---|
| Environment（环境） | 提供执行环境抽象 | Docker、E2B、Daytona、Local 本地宿主 |
| Executor（Agent 运行时） | 接入不同 Agent 执行栈 | 通过 ACP 接入通用 coding agent（pi、Codex、Claude Code、OpenCode 等），并原生收录 DeepSeek 官方 dsh、NVIDIA nooa、SWE-agent miniswe |

切换执行环境或待测 Agent 只需修改 `profiles.yaml` 中的对应声明，原有 dataset 无需任何改动即可直接运行。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

这一层与 [[concepts/agent-sandbox|Agent 沙箱]] 的隔离设计直接相关：环境插件把「本机 / 容器 / 云沙箱」抽象成同一接口，使「同一评测任务跨环境可迁移」成为默认能力，而不是每个项目重写一遍适配层。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

## CLI + Skills：让 coding agent 驱动评测流程

ageval 附带专用 CLI 与 skills，安装后 coding agent 就能理解 ageval 的 CLI 指令与数据结构规范，被直接「吩咐」完成原本繁琐的工程操作：自主编写新 benchmark（自动生成各 task 所需的 `run.py` 与 `evaluator.py`）、把团队现有评测脚本迁移为标准 ageval dataset、自动拉起测试环境跑完评测矩阵并汇总不同配置的对比分析报告。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

```bash
uv tool install ageval-cli
npx skills add ZJU-REAL/ageval
# 单次运行的最小示例
uv run ageval run examples/datasets/minimal-demo --task terminal-jsonl-agg
# 本地启动轨迹复盘查看器
uv run ageval view examples/datasets/minimal-demo
```

本地 `ageval view` 会启动轻量 Web 轨迹查看器，按 **Jobs → Tasks** 层级复盘每次运行：展示 environment / run / evaluate 各阶段精确耗时；逐轮展开 Agent 输入、Tool Calls、终端输出与模型回复；每个失败 task 附带完整重跑命令，可在终端单独复现与单步调试。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

「CLI + skills 让 agent 自己写 benchmark / 迁移评测集」是把 [[concepts/coding-harness-engineering|Coding Harness 工程]] 的思路反向用在评测基建自身——评测流程成为 agent 可编排的对象。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

## Hub：Leaderboard / Agents / Models 三视图

ageval Hub 把评测结果按三个维度组织，核心是**环境与配置严格对齐的对比**：

- **公开榜单（Leaderboard）**：每一项成绩都绑定具体的 Agent 运行时版本与执行环境（如 Docker、E2B）；更换底层沙箱或调整工具调用策略后，得分与资源消耗的变化可横向对照；每次提交附不可变 `lock.json` 与完整轨迹文件，其他开发者可拉取配置一键复现。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]
- **Agents 市场**：团队可把调优好的 Agent 方案（Prompt 模板、Tool 定义、底层插件依赖声明）打包发布为 Agent 包，他人运行时只需指定 `--agent <org/name@version>` 即可在任意 dataset 上开箱使用。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]
- **Models 视图**：以模型为主维度查看其在各类 dataset 与 Agent 组合下的解题成功率和调用成本；也可固定同一套 Agent 运行时（相同 Prompt 策略与工具调度链），横向对比不同模型的代码生成质量与通过率。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

「固定 Agent 架构、只换模型」与「固定模型、只换 Harness」这两种对照，正面回应了评测中**模型能力与 Harness 能力互相混淆**的问题——这与 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 里强调的「评测面（eval surface）要显式声明」是同一诉求。^[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026.md]

## 与既有评测方案的定位差异

ageval 与已收录的评测工具/方法论形成互补而非重复：

- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|AWS agent-evalkit]] 同样走「开源 CLI 评测工具包」路线，但 ageval 的差异化在**环境 × Agent 运行时二维插件矩阵**与不可变证据链；
- [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop|从指标到闭环的系统化评测指南]] 关注评测指标体系与闭环流程，ageval 提供的是承载这套流程的执行底座；
- [[entities/agent-evaluation-turing-meituan-2026|美团 Turing 评测]] 与 [[entities/agent-evaluation-four-layer-outcome-decision-action-reliability-aliexpress-2026|阿里四层评测框架]] 侧重评测维度建模，ageval 侧重「让不同维度、不同运行时下的结果可比且可复现」。

## 深度分析

换运行时导致评测结果不可比，本质是评测面未显式声明——模型能力与 Harness 能力混在同一个分数里。ageval 把执行环境与 Agent 运行时都做成可插拔插件，同一份 dataset 可在任意组合下重跑，等于把「评测面」从论文脚注升格为一等配置项：每项成绩绑定运行时版本与执行环境，混淆变量在基础设施层就被消除。这对可复现性是质变——复现者不再逆向猜测榜单背后的脚手架，拉取 lock.json 与轨迹文件即可重放。

### 服务契约：一次接口稳定性的提前投资

export/inject 声明式契约加 requires⊆capabilities 静态校验，是「用接口约束换长期演进自由」的典型交易：基座代码不含任何环境或 Agent 特定分支，接入 DeepSeek dsh 零源码改动，只提交一份插件声明。代价是契约需一次设计到位——capabilities 划分失当会被所有插件放大；收益是每接入一个新插件都在增值生态而非加深耦合，这与 [[concepts/coding-harness-engineering|Coding Harness 工程]] 中「以稳定原语承载多变上层」的思路同构。

### 确定性流水线与 finally 清理：评测基建的卫生学

lock→environment→run→evaluate→record 单向流水线把失败前置：lock 阶段静态拦截依赖与凭证问题，评分收敛在 evaluator.py 单点，cleanup 钩子挂在 finally，成功、失败、超时、中断四条路径都回收容器、注销网络、擦除凭证。这类卫生细节在 demo 里看不见，却决定评测系统能否长期无人值守运行——残留容器烧钱，残留凭证是安全事件。gold 延迟上传与打分容器断网同理：防泄题不靠纪律，靠让泄题在结构上不可行。

### 不可变证据链：把复现从声明变成机制

每次运行自动封存 lock.json（执行配置与拓扑快照）、result.json（评分细节）与 trajectory.jsonl（完整交互轨迹），Hub 上每项提交都携带这套证据，他人拉取配置即可一键复现。复现性因此不再是作者的自述，而是由归档格式保证的机制：证据链由流水线产出，与结果同生共死，无需事后补记；轨迹查看器按 Jobs→Tasks 逐轮回放，失败 task 附带重跑命令，复现与调试共用同一份数据。

### 与 SWE-bench 式黑盒 Harness 的对照

不少公开 benchmark 只公布模型名与分数，不说明用了哪套 Harness、不标沙箱版本与 Prompt 模板，外界难以复现——[[entities/programbench-swe-agent-benchmark|ProgramBench / SWE-agent 基准]] 一类以任务Validity为核心的基准（环境细节以原项目文档为准）若不绑定执行配置，跑分与复现就是两件事。ageval 反其道而行：成绩绑定运行时与环境，证据链随运行自动归档，复现是默认产出而非额外承诺。评测基建由此从一次性跑分脚本变成可审计的公共设施，正是 [[concepts/evaluation-harness-design|评测 Harness 设计]] 主张的显式评测面；对评测面随时间退化的担忧，则可与 [[concepts/eval-surface-rotation|评测面轮换]] 的视角互补。

## 实践启示

1. 报告评测结果时同时声明 Agent 运行时版本、执行环境与关键配置——缺这些元数据，分数既无法横向比较，也无法复现。
2. 评测系统优先做静态前置校验（依赖图、凭证、能力匹配），把失败提前到最便宜的阶段，而非跑到一半崩溃。
3. 用声明式契约（export/inject 加 requires⊆capabilities）替代硬编码分支：新环境、新 Agent 以插件接入，基座零改动。
4. 资源清理必须挂 finally：容器、网络、临时凭证在成功与失败路径上都要回收，这是无人值守评测的前提。
5. 参考答案在评分前对被测环境不可见，打分容器断网——防作弊靠结构隔离，不靠提示词纪律。
6. 让 coding agent 经 CLI 加 Skills 编排评测（写 benchmark、迁移数据集、汇总对比报告），评测基建随之成为 agent 可日常操作的对象。

## 相关实体

- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]
- [[concepts/agent-sandbox|Agent 沙箱]]
- [[concepts/coding-harness-engineering|Coding Harness 工程]]
- [[concepts/harness-engineering|Harness Engineering]]
- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|AWS agent-evalkit]]
- [[entities/deepseek-harness-cordis-plugin-runtime-tencent-2026|DeepSeek Harness 插件运行时]]
- [[entities/programbench-swe-agent-benchmark|ProgramBench / SWE-agent 基准]]

→ [[raw/articles/ageval-pluggable-agent-eval-plugin-decoupling-zju-2026|原文存档]]
