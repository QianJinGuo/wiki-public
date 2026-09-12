---
title: "小米 Harness 工程：从个人实践到团队标准的 Prompts→Hooks→Plugin 三次跨越"
type: entity
tags: [xiaomi, harness-engineering, ai-coding, team-practice, quality-gate, hook, plugin, claude-code, enterprise-practice]
created: 2026-07-29
updated: 2026-09-12
rating: v9c9
sources:
  - raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 小米 Harness 工程：从个人实践到团队标准

小米技术团队关于 Harness Engineering 从个人实践到团队标准的系统化工程方案。核心洞察：**提示词是建议，Harness 让规则落地。** ^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

> → [[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin|原文存档]]

## 三次跨越

### 第一次：把流程写进 Prompt
用 /flow 命令标准化开发环节，结构化 Prompt 模板约束 AI。根本局限：Prompt 是"软"约束，AI 绕过规则、上下文压缩后"忘记"约束、无法核验操作。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

### 第二次：把约束下沉到 Hook
在 Claude Code PreToolUse / PostToolUse 生命周期挂载检查脚本。首次具备可执行的流程强制力。但系统"单体"——配置分散在各项目，靠 rsync/git pull 分发。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

### 第三次：把约束沉淀为可复用能力（插件化）
管控逻辑封装为 Claude Code 插件，通过内部 Marketplace 分发。三重故障闭锁：Bash 状态守卫 + 源码守卫 + 状态文件防篡改。门禁引擎（workflow-gate.sh，约 2600 行）统一验证器完成 5 路检查（技能事件/CLI 事件/工件/度量/项目状态）。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

## 双通道证据机制

任何关键步骤同时留存两份独立证据：
- **工程工件**：AI 在文档中记录技能、完成时间等
- **工具执行记录**：Hook 自动写入事件记录，不依赖 AI 主动记录

验证时同时检查两个来源。两类证据同时满足，当前阶段才被认为真正完成。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

## 质量门禁三层拦截

1. **PreToolUse 源码守卫**：方案阶段阻止修改业务代码（exit 2 = 阻断工具调用），强制"方案未定，代码不动"
2. **PreToolUse Commit 守卫**：前置步骤未完成无法提交
3. **Bash 状态守卫**：审查命令行，仅放行白名单只读操作；写入 .workflow-state.json 统一通过 mark-step 子命令^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

## 实测数据（2026-04-24 至 06-05）

| 指标 | 数值 |
|------|------|
| 业务项目 | 3 个 |
| 开发者 | 6 名 |
| 正式需求 Flow | 23 个（21/23 未跳过步骤，合规率 91.3%） |
| Flow 首尾耗时中位数 | 约 23.4 分钟 |
| Flow 首尾耗时均值 | 约 38 分钟 |
| 知识类 Markdown | 110 份（首次审计发现 15 项高风险错误） |
| 开发时间节省 | 约 40%-50%（团队经验口径） |
| Brainstorm 耗时 | 25min → 12min（模板约束后） |

^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

## 案例：GP Inline Install

完整 /flow 案例（brainstorm → propose → implement → verify）：减少 3 轮评审修改，减少 2 个功能缺陷。关键决策：brainstorm 阶段"安装时机"初稿默认"应用启动时"，模板强制至少 2 个备选方案，改选"用户首次触发广告场景时"——避免后续 2 个功能缺陷。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

## 诚实边界

- **Bus Factor = 1**：整个约束引擎是一人维护的 2600 行 Bash
- **平台锁定风险**：深度依赖 Claude Code Hook API
- **标准流程偏重**：低风险小改动需轻量路径
- **因果链未建立**：合规率到业务结果的归因仍在积累
- **最小可行版本**：50 行脚本→一个状态文件+一个拦截器

^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md]

## 深度分析

### 一、三次跨越的本质：规则载体从语言到代码再到分发单元

Prompt、Hook、Plugin 常被读成功能的三级叠加，但从工程视角看，它们是同一条规则在三种"载体"上的迁移：第一次把流程写进 Prompt，规则以自然语言存在；第二次下沉到 Hook，规则以可执行脚本存在；第三次封装为插件，规则以可安装的分发单元存在。载体每换一次，规则的"硬度"就上升一级——语言可以被解释也可以被忽略，代码只能被执行或被阻断，而分发单元则决定这条规则能覆盖多少人、能否版本化演进。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md:13-30]

### 二、为什么 Hook 比 Prompt 更能"执行"规则

Prompt 作为约束有三个结构性失效点：模型可能绕过规则、上下文压缩后可能"忘记"约束、且无法核验 AI 的实际操作。三者都不是"提示词写得更好"能修复的，因为它们同源——约束依赖模型的主动配合。Hook 把检查点移到了工具调用的边界：执行前校验阶段与准入条件，不满足即用 exit 2 阻断工具调用；执行后自动记录事件。规则不再依赖模型"记得"，而取决于运行时是否放行，从"建议"变成"闸门"，这也是 [[entities/agent-hooks-programmable-workflow|Hook 可编程工作流]] 被反复强调的原因。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md:13-30]

### 三、门禁要分层，而不是在同一个点上反复校验

三层拦截各自守的是不同的失败模式，而非对同一动作的重复校验：PreToolUse 源码守卫守"方案/代码"边界，强制"方案未定，代码不动"；PreToolUse Commit 守卫守"流程完整性"边界，前置步骤未完成不允许提交；Bash 状态守卫守"状态可信"边界，只放行白名单只读操作，受保护状态只能经统一接口更新。真正值得借鉴的是这种切分方式——先枚举会出错的边界，再为每个边界配一个拦截器；若三层都在拦同一件事，那就是设计冗余而非纵深防御。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md:40-44]

### 四、双通道证据：让 AI 的"声明"必须被外部事件背书

双通道证据机制同时留存工程工件（AI 在文档中记录技能、完成时间等）与工具执行记录（Hook 自动写入事件，不依赖 AI 主动上报），验证时两个来源都必须满足。它消解了单边证据的两种失真：只有工件没有工具事件，说明 AI 声称完成但实际没有执行；只有工具事件没有工件，说明动作发生了但产物不完整。本质是把"AI 是否真的做了"从自我报告变成可外部核验的事实——这正是 [[entities/harness-engineering|Harness Engineering]] 相对 Prompt 的核心增量。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md:32-38]

### 五、从个人实践到团队标准，真正的摩擦在分发与维护

技术方案一旦要成为团队标准，瓶颈就从"规则怎么写"转到"规则如何送到每个人手上、由谁维护"。第二次跨越的 Hook 系统是"单体"的，配置分散在各项目本地目录，靠 rsync/git pull 分发，正因如此才需要第三次插件化、经内部 Marketplace 统一分发。代价也随之显形：Bus Factor = 1（整个约束引擎是约 2600 行 Bash 由一人维护）、深度依赖 Claude Code Hook API 的平台锁定、标准流程对低风险小改动偏重。团队标准能否持续，最终取决于它是否可被多个人而非一个人演进。^[raw/articles/xiaomi-harness-engineering-prompt-to-hook-to-plugin.md:59-65]

## 实践启示

1. **先判断规则的"硬度"需求**：只写在 Prompt 里的流程，得到的永远是建议；若规则必须被遵守，就要把它下沉为工具边界上的可执行拦截。
2. **为每个失败边界配一个门禁，而不是堆校验**：方案/代码、流程完整性、状态可信是三类不同边界，分层拦截才形成纵深，重复校验只是冗余。
3. **关键步骤用双通道留证**：让"完成"必须同时有工程工件与工具事件背书，杜绝 AI 自证的"声称完成"。
4. **约束从第一天就按可分发单元设计**：单体 Hook 靠 rsync/git pull 分发无法规模化，插件化加内部分发渠道才是把实践推成团队标准的前提。
5. **为低风险改动留一条轻量路径**：标准流程偏重是小改动场景最主要的抵触来源，缺少快车道会让开发者绕过整套机制。
6. **从一个最小可行拦截器起步**：约 50 行脚本、一个状态文件、一个拦截器即可验证机制，再逐步长成完整的门禁引擎。

## 相关实体

- [[entities/xiaomi-harness-engineering-jdk-upgrade|小米 JDK21 升级中可控演进的 AI 工程实践]] — 同团队 JDK 升级实践（本专栏第 5 期）
- [[entities/harness-engineering|Harness Engineering]] — 通用 Harness 工程概念
