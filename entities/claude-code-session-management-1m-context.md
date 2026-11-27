---

title: "Claude Code Session 管理与 1M 上下文最佳实践"
created: "2026-05-09"
updated: 2026-11-27
type: "entity"
tags: [claude-code, session-management, context-management, context-rot, compaction, subagent, rewind]
review_value: 8
review_confidence: 7
sources:
  - raw/articles/claude-code-session-management-1m-context
related:
  - "concepts/claude-code-deep-architecture-analysis"
  - "concepts/gsd-get-shit-done-context-management-tool"
  - "entities/agent-harness-context-management-working-set"
  - "concepts/hermes-agent"
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

## 核心洞察：每轮对话都是一个分叉决策点
每次 Claude 完成一轮对话，用户在发送下一条消息前，有五个选项构成上下文管理的核心工具集：   ^[raw/articles/claude-code-session-management-1m-context.md]
| 操作 | 本质 | 适用场景 |
|------|------|----------|
| **继续（Continue）** | 自然延伸 | 同一任务连续性工作时 |
| **回退（/rewind / 双击 Esc）** | 回到分支点重新发指令 | 方向错误时，比修正更优 |
| **清除（/clear）** | 手动写简报开新会话 | 需要完全控制上下文转移时 |
| **压缩（/compact）** | 让模型总结会话后继续 | 会话变长但仍有信息需要保留时 |
| **子智能体（Subagents）** | 独立干净上下文执行子任务 | 中间输出大量的独立子任务 |

## 核心策略
### 1. 开启新任务的时机
**经验法则：当你开始一项新任务时，就应该开启一个新会话。** 1M 上下文让长任务更可靠（如从零构建全栈应用），但关联任务（如给刚实现的功能写文档）可能需要保留部分上下文。 ^[raw/articles/claude-code-session-management-1m-context.md]

### 2. 回退优于修正
这是最重要的上下文管理习惯。当 Claude 尝试某方法失败时：^[raw/articles/claude-code-session-management-1m-context.md]


- ❌ **本能反应**："那没用，试试方法 X" — 失败的步骤仍留在上下文中继续污染注意力
- ✅ **正确做法**：回退到读取文件之后的那一刻，重新发指令，结合刚学到的教训
还可以使用"从此处总结（summarize from here）"让 Claude 生成交接消息。 ^[raw/articles/claude-code-session-management-1m-context.md]

### 3. 压缩 vs 清除
- **Compact（压缩）**：有损操作，信任 Claude 决定哪些信息重要。可通过指令引导（如 `/compact 重点关注 auth 重构，丢掉测试调试的部分`）
- **Clear（清除）**：手动提炼重点后重新开始，更费力但完全可控

### 4. 糟糕的压缩
当模型无法预测用户工作方向时，常发生糟糕的压缩。例如：漫长的调试后自动压缩总结了排查过程，但用户下一条消息是修复另一个警告，而该警告已在摘要中被丢弃。 ^[raw/articles/claude-code-session-management-1m-context.md]
**对策**：1M 上下文给了更充裕的时间，可以根据接下来的计划主动运行带描述的 `/compact`。 ^[raw/articles/claude-code-session-management-1m-context.md]

### 5. 子智能体作为上下文隔离工具
子智能体拥有独立的、干净的上下文窗口，适合以下场景：^[raw/articles/claude-code-session-management-1m-context.md]


- 验证工作成果（基于 spec 文件）
- 研究其他代码库的实现方式
- 为 git 改动编写文档
**心理测试标准：以后还需要这些工具的原始输出吗？还是只需要结论？**^[raw/articles/claude-code-session-management-1m-context.md]


## 与相关概念的关联
- [[concepts/claude-code-deep-architecture-analysis|Claude Code 架构深度分析]] — 架构上下文压缩机制的源码级实现
- [[entities/agent-harness-context-management-working-set|Agent Harness 上下文管理：工作集视角]] — 上下文≠聊天记录，工作集视角下的四框架对比
- [[concepts/hermes-agent|Hermes Agent]] — 开源 Agent 的上下文管理策略对比

## 参考
- [[raw/articles/claude-code-session-management-1m-context.md|原文存档]]

## 相关实体
- [[entities/claude-code-subagent-context-hygiene|Claude Code Subagent 上下文卫生]]
- [[entities/claude-code-prompt-context-harness|深度解析 Claude Code 在 Prompt / Context / Harness 的设计与实践]]
- [[entities/context-window-management|Agent 上下文窗口管理对比]]
- [[entities/agent-context-management-architecture-patterns|Agent 上下文管理工程模式收敛 — 多框架代码级横向对比]]

- [[entities/claude-code-openclaw-memory-comparison|Claude Code vs OpenClaw Agent 记忆系统对比]]

## 深度分析

### 分叉决策点：会话即状态机

原文最具结构性的洞察是把每一次对话轮次的结束都视为一个**分叉决策点**：上下文中已经累积了系统提示、历史指令、工具调用及其输出，此刻选择下一步操作，本质上是在对上下文状态做不同类型的干预。^[raw/articles/claude-code-session-management-1m-context.md] 五个选项对应五种状态转移：Continue 是保持状态自然延伸；/rewind 是回滚——跳回任意一条历史消息并**删除该点之后的全部内容**，从那里重新下发指令^[raw/articles/claude-code-session-management-1m-context.md:28-33]；/clear 是重置，通常携带手工提炼的简报开新会话^[raw/articles/claude-code-session-management-1m-context.md:30-33]；/compact 是让模型总结当前会话并以摘要替换历史^[raw/articles/claude-code-session-management-1m-context.md:31-33]；Subagent 则是把下一块工作外包给一个拥有独立干净窗口的代理，只把最终报告取回父上下文^[raw/articles/claude-code-session-management-1m-context.md:32-33;57-60]。Continue 是最自然的默认反应，但真正构成上下文管理杠杆的恰恰是其余四个选项^[raw/articles/claude-code-session-management-1m-context.md:26-33]。

### 回退与压缩：两种"减重"的机制差异

回退之所以优于"告诉它换种方法"，机制在于注意力经济：失败的尝试路径若留在上下文中，会持续占用注意力预算；而 rewind 把失败路径整体移除，再结合刚学到的教训重新发指令，相当于回到决策点重做选择，而非在错误状态上追加否定性指令。^[raw/articles/claude-code-session-management-1m-context.md:40-44] 原文还提供了一种折中："从此处总结"（summarize from here）让模型把自己的踩坑经验压缩成一条交接消息——被形容为"未来的 Claude 给过去的 Claude 留的小便条"。^[raw/articles/claude-code-session-management-1m-context.md:42-44]

压缩与清除则是两种逻辑相反的减重方式。Compact 是有损操作：信任模型决定哪些信息重要，可以用指令引导其取舍（如"重点关注 auth 重构，丢掉测试调试的部分"）；Clear 是手工提炼后干净起步，更费力但上下文完全可控。^[raw/articles/claude-code-session-management-1m-context.md:46-49] 糟糕的压缩发生在模型无法预测用户工作方向时：长调试后触发的 autocompact 倾向于总结排查过程，若用户下一条消息转向另一个不相关警告，该警告可能已被摘要丢弃。更棘手的是，受上下文腐化影响，模型执行压缩时恰恰处于其能力的最低点。^[raw/articles/claude-code-session-management-1m-context.md:51-55] 这说明压缩质量与压缩时机耦合：压缩本身既消耗上下文质量，又在质量最差时执行。

### 1M 上下文改变了什么：更大的预算，而非无限记忆

1M 窗口的构成要素并未变化——系统提示词、全部对话、每次工具调用与输出、读过的每个文件都在其中；上下文腐化同样未消失：注意力仍会随 token 增长被摊薄，陈旧内容仍会干扰当前任务。^[raw/articles/claude-code-session-management-1m-context.md:21-24] 真正变化的是三类权衡：

1. **任务可行性**：长任务（如从零构建全栈应用）变得可靠可做。^[raw/articles/claude-code-session-management-1m-context.md:36-38]
2. **检索 vs 全量塞入**：关联任务（如为刚实现的功能写文档）不必强行开新会话——新会话必须重读文件，更慢且更贵；保留部分相关上下文反而是一种成本更优的"检索"。^[raw/articles/claude-code-session-management-1m-context.md:36-38] 反之，无关联内容仍应通过简报、spec 文件等方式桥接，而不是依赖长上下文全部携带。
3. **压缩时机自主权**：更充裕的窗口意味着可以按接下来的计划**主动**运行带描述的 /compact，而不是被动等待 autocompact 在不可预测的时点做出糟糕取舍。^[raw/articles/claude-code-session-management-1m-context.md:52-55]

换言之，1M 提高的是注意力预算的上限与容错空间，但"预算有限"这一前提不变——把 1M 当作免压缩借口，只会加速腐化。

### 会话卫生的实操模式

综合原文，可沉淀为一组可复用的卫生模式：新任务开新会话是默认项，例外仅限上下文确有保留价值的关联任务^[raw/articles/claude-code-session-management-1m-context.md:36-38]；把 Subagent 判断前移到任务规划阶段而非会话变脏之后，判据是心理测试——"以后还需要工具的原始输出吗，还是只需要结论"——只需结论即适合隔离^[raw/articles/claude-code-session-management-1m-context.md:58-60]，典型用例包括按 spec 文件验证成果、研究其他代码库的实现方式、根据 git 改动编写文档^[raw/articles/claude-code-session-management-1m-context.md:62-64]；每个轮次结束后主动评估分叉选项而非默认 Continue，这是普通用户与高级用户的核心分野^[raw/articles/claude-code-session-management-1m-context.md:65-67]。这与 [[entities/claude-code-subagent-context-hygiene|Subagent 上下文卫生]] 的隔离实践一脉相承，也可在 [[entities/agent-harness-context-management-working-set|工作集视角]] 下理解为：维持工作集精炼，把历史与结论外置到可检索的工件中。

## 实践启示
1. **在每个回复后强制进行分叉决策**：不要默认点"继续"，而是主动评估是否需要rewind/clear/compact/subagent——这是区别普通用户和高级用户的关键习惯^[raw/articles/claude-code-session-management-1m-context.md]
2. **建立"交接消息"机制**：使用"summarize from here"生成交接文档，让未来的自己或新的会话能够快速接续当前上下文^[raw/articles/claude-code-session-management-1m-context.md]
3. **主动Compaction优于被动**：根据接下来的计划主动运行带描述的compaction，而非等待autocompact触发——这需要培养对任务走向的预判能力^[raw/articles/claude-code-session-management-1m-context.md]
4. **新任务开新会话是老生常谈但仍被低估**：关联任务可以通过spec文件桥接，而非依赖长上下文保持——这是架构思维而非省事思维^[raw/articles/claude-code-session-management-1m-context.md]
5. **Subagent使用反向训练**：不要等到会话变脏才想起Subagent，而是从任务规划阶段就判断是否需要隔离——这需要建立"任务拆分的上下文边界思维"^[raw/articles/claude-code-session-management-1m-context.md]