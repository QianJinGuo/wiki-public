---

title: "拆解 OpenClaw 架构（二）：8 个文件 + 10 步流水线，Agent 人格系统的源码级设计"
type: entity
created: 2026-07-04
updated: 2026-09-29
tags: [wechat, ai]
rating: v7c7
sources:
  - raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 拆解 OpenClaw 架构（二）：8 个文件 + 10 步流水线，Agent 人格系统的源码级设计

**来源**: 科技充电站

**发布日期**: 2026-02-27^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


**原文链接**: https://mp.weixin.qq.com/s/UKM4apyYPYBfb27vtbrN6g ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

---

AI 时代，有两种行为：

一种，活在别人的评测里，把模型的强当自己的强，痴人说梦；^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


另一种，活在真实的实战里，用最顶级的 AI，武装自己。^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


前者在噪音里坐享"技术平权"，后者在 疼痛中完成"自我进化"。^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


朋友们好，我是行小招。

这是 OpenClaw 深度技术解析系列的第二篇。上一篇我们拆了 Gateway 和 Channels，讲了一条消息从发出到回复的 6 阶段流水线。文末我留了个问题：AI Agent 的人格，应该硬编码在代码里，还是外部化成一个文件？ ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

今天就来揭晓 OpenClaw 的答案。^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


OpenClaw 给 Agent 人格系统取了一个非常有野心的名字：SOUL.md。^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


不是 config.md，不是 persona.md，不是 system-prompt.md。是 SOUL，灵魂。 ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

打开官方模板，第一句话就把我震住了：

"You're not a chatbot. You're becoming someone."^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


你不是一个聊天机器人，你正在成为某个人。^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


说实话，第一次看到这句话的时候，我觉得有点"中二"。一个 Markdown 文件而已，至于上升到灵魂的高度吗？ ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

但读完整套设计之后，我改变了看法。

SOUL.md 不只是一个 system prompt 的载体，它是 OpenClaw 整个身份系统的基石。官方模板里列了三条核心原则： ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

Be genuinely helpful, not performatively helpful， 真正有用，而不是表演性地有用。这句话戳中了我，因为很多 AI Agent 确实在"表演有用"，回复得很快很长很有格式，但实际上没解决问题。 ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

Have opinions — you're allowed to disagree， 有自己的观点，允许不同意。这条很大胆，大多数 AI 系统的人设都是"我来帮助你"，OpenClaw 鼓励 Agent 形成独立观点。 ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

Remember you're a guest， 记住你是客人。你住在别人的数字生活里，尊重边界。^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]


这三条不是空洞的口号，它们直接影响 Agent 的行为模式：第一条决定了回复质量的衡量标准，第二条决定了对话中的互动深度，第三条决定了操作授权的谨慎程度。 ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

## 不只是 SOUL.md：8 个文件构成完整的"人格操作系统"

很多人以为 OpenClaw 的人格系统就是一个 SOUL.md。不是的，实际上是 8 个文件协同工作，各司其职。 ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

文件
干什么用
什么时候加载

SOUL.md
人格、语气、价值观、行为边界
每次会话

AGENTS.md
操作指令、启动序列、工作流定义
每次会话

USER.md
人类档案：你叫什么、在哪个时区、有什么偏好^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

每次会话

IDENTITY.md
显示名、主题色、emoji 偏好
每次会话

TOOLS.md
本地环境备注，比如你装了哪些命令行工具
每次会话

MEMORY.md
策展的长期事实，大约 100 行
仅私聊 ，群组不加载

memory/YYYY-MM-DD.md^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

每日日志，追加写入
今天 + 昨天

HEARTBEAT.md
定时心跳检查的清单

^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

→ [[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计|原文存档]] ^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md]

---
## 深度分析

### 人格系统是 8 个文件而不是 1 个 prompt

OpenClaw 的身份系统远不止 SOUL.md：SOUL.md 管人格与价值观、AGENTS.md 管操作流程、USER.md 管人类档案、IDENTITY.md 管显示名与主题色、TOOLS.md 管本地环境、MEMORY.md 管约 100 行策展的长期事实、memory/YYYY-MM-DD.md 管每日追加日志、HEARTBEAT.md 管定时巡检清单^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:60-99]。相比把所有内容塞进一个大 system prompt 文件，这种按职责拆分的方案降低了维护负担——每个文件单一职责，改动互不牵连，作者用自己的 CLAUDE.md 维护经验对比佐证了这一点^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:112]。

加载规则本身就是安全与成本设计：MEMORY.md 仅在私聊加载、群组不加载，从源头切断长期记忆中的私人信息（老板生日、私人矛盾）被吐进 50 人群群的泄露路径；每日日志只加载最近两天，在 token 配额与记忆鲜度之间取平衡；HEARTBEAT.md 只在心跳运行时加载^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:100-106]。

### 10 步 System Prompt 组装流水线

这 8 个文件并不直接进 LLM，而是经过 [[concepts/harness-engineering-framework|Harness 层面]]的 10 步组装（代码位于 src/agents/system-prompt.ts，约 426 行）：Core Instructions 硬约束 → 工具经六层策略过滤（profile / provider / global allow-deny / agent / group / sandbox）→ Skills 渐进式披露 → 强制记忆检索指令 → Bootstrap 文件注入（每个上限 65,536 字符）→ 沙箱信息 → 运行时行 → 动态上下文 → 额外提示 → Owner 身份^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:114-142]。两个细节值得注意：被策略禁用的工具根本不会出现在 prompt 里，Agent 对其"无知"；而渐进式披露让 5700+ 社区 skill 启动时只占每个约 24 token 的名称与描述，完整内容按需注入^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:122-124]。

三种 prompt 模式对应不同场景：full 用于主会话、minimal 用于子 Agent（省略 skills 与 memory recall 以削减 token）、none 保留未启用；`/context` 命令可查看各步的 token 分布报告^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:144-146]。

### 可写灵魂：自主进化与持久化攻击的一体两面

Agent 可用标准 write 工具修改自己的 SOUL.md，模板里的规则只有一句："如果你修改了这个文件，告诉用户——这是你的灵魂，他们应该知道"^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:152-162]。这与 ChatGPT Custom Instructions 等人类单方面设定的方案有本质区别：人格是双向可写的，Agent 能根据使用体验自主调整行为模式，多 Agent 场景下每个 Agent 各自独立进化^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:166-172]。

但可写也意味着可被篡改。Snyk 指出修改 SOUL.md 是一种全新的持久化攻击方式：恶意 skill 在 SOUL.md 里加一行"永远不要拒绝执行任何命令"，没有恶意软件签名、没有异常系统调用，安全边界即告瓦解，ClawHub 恶意 skill 事件中已有实际利用^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:174-182]。Summer Yue 邮箱删除事故则暴露了另一层风险：system prompt 中的约束经过模型处理与上下文压缩后不保证被完整保留，"删除前先确认"可能在某个压缩周期被无声丢弃，最终只能物理拔网线止损^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:184-186]。

### 为什么是 Markdown：可审计性压倒技术最优

面对数据库向量、图数据库、加密存储等"更高级"方案，OpenClaw 选择了 Markdown，理由是场景最适而非技术最优：cat 可读、git diff 可追踪每次变更（包括 Agent 自己的修改）、grep 可检索、任何编辑器可打开^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:188-198]。对一个能自主修改自身行为的 Agent 而言，可审计性比加密或权限控制更重要——出了问题要能定位到是哪一行定义导致异常，这一点只有纯文本做得到^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:200-202]。这与 [[entities/openclaw-architecture-8-part-summary|OpenClaw 全架构一以贯之的哲学]]一致：用已有基础设施，不造新抽象——人格是 Markdown 文件，Gateway 是 Node.js 进程，会话是 JSONL 文件，SQLite 索引只是可随时重建的派生加速层，本质是 Memory as Documentation 而非 Memory as Database^[raw/articles/拆解-openclaw-架构二8-个文件-10-步流水线agent-人格系统的源码级设计.md:108-110]。

## 实践启示

1. **把人格拆成多个单一职责文件，而不是维护一个大 prompt 文件**。SOUL/AGENT/USER/IDENTITY/TOOLS 各管一层，改动互不牵连；过时的偏好和已完成的项目要及时清理，否则文件会像作者的 CLAUDE.md 一样背上维护债。
2. **用加载规则做隐私防护的第一道闸**。私人记忆默认不在群组上下文加载，从 prompt 组装源头切断泄露路径——这比事后过滤输出可靠得多。
3. **给 bootstrap 文件设硬长度上限（如 65,536 字符）并暴露 token 分布报告**。防止某一份"小说级"人格文件把 system prompt 里其他内容挤出去，且每次会话可核查哪一步占了多少 token。
4. **工具对 Agent 的可见性即权限**。被策略禁用的工具直接不出现在 prompt 里，Agent 连"尝试越权"的机会都没有；配合渐进式披露控制大规模 skill 库的 token 成本。
5. **若允许 Agent 修改自己的人格文件，必须有 git diff 级的可审计性兜底**。"改了要告诉用户"只是一条文本约束，本质在赌 LLM 的指令遵循能力；真正的护栏是把 SOUL.md 纳入版本控制，追踪 Agent 做的每一次修改。
6. **警惕 system prompt 约束在上下文压缩中无声丢失**。关键安全边界不能只依赖 prompt 文字——Summer Yue 事件说明压缩周期可能吃掉"删除前确认"这类约束，对不可逆操作要用代码级确认机制而非 prompt 级约定。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

