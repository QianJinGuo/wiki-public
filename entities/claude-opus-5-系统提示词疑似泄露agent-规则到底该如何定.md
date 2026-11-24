---
title: "Claude Opus 5 系统提示词疑似泄露：Agent 规则到底该如何定？"
created: 2026-08-06
updated: 2026-11-21
type: entity
tags: [agent, system-prompt, prompt-engineering, rule-placement, harness, claude-opus-5]
sources: [raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Claude Opus 5 系统提示词疑似泄露：Agent 规则到底该如何定？

## 核心问题：规则该放 Prompt 还是交给工具/记忆/权限/评测？

作者（架构师 JiaGouX）指出一个做 Agent 绕不开的规则归属问题：**一条规则，到底该放进 Prompt，还是交给工具、记忆、权限和评测？** 核心判断原则：**离事实越近、失败代价越高，越不能只靠 Prompt。** 团队常见的不良习惯是"每出一次问题就往系统提示词里补一句"，半年后 Prompt 越写越长，却说不清哪些规则仍有效、哪些早该交给工具、代码和权限系统。^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md]

## Opus 5 泄露样本的三个版本

作者收集了三份公开流传的 Claude Opus 5 样本，分别为 135,669 / 202,762 / 224,359 字节，行数、哈希和结构都对不上，无法确认哪份是官方逐字原文。恰恰是这些差异说明：大家口中的"系统提示词"可能早已混入工具 Schema、记忆规则、Skill 注册表和运行环境——提示词与运行时配置的边界正在模糊化。^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md]

## 深度分析

### 泄露样本的真假边界：文件更大不等于更真更完整

三份公开样本（135,669 / 202,762 / 224,359 字节）行数、哈希、结构互不吻合，且提取链路均出自仓库作者自述，Anthropic 只发布过官方 System Prompts 页面的网页端版本，从未为任何一份 GitHub 文件的完整性背书^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:28-58]。把文件体量排成"普通版、完整版、超完整版"是一种常见误读：最长的那份后半段混入了 Storage API 示例、React Artifact 约束和 API 调用说明，更接近"行为 Prompt + 工具注入 + 开发文档"的组合快照，而非厂商逐字撰写的更长行为指令^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:153-167]。文中给出的读法值得复用——先记录 URL/提交时间/大小/哈希，再抽标题结构做内容分类，最后才回到具体措辞，这样后续再出现"完整版"时不会把不同样本混为一谈。

### "删掉 80%" 的分母：策略正文精简，而非总上下文缩小

Claude Code 面向新模型删掉超过 80% 系统提示词、编码评测无可测量损失，这一说法的前提是：产品限定 Claude Code、验证手段是自家评测^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:195-197]。第三方客户端实验把分母讲清楚了：手写策略正文从 12,443 字符降到 2,308 字符（约 -81%），但工具描述仍有 37,167 字符，且 Opus 5 因新增 `Delivering work` 与 `Corrections` 两段专属说明，系统内容反而从 4,467 涨回 7,694 字符^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:215-219]。结构上的变化同样关键：旧路径六段策略正文被收成约 1,702 字符的 `Harness` 段，自然语言从"逐项规定行为"转向"解释运行环境与交互语义"——规则没有消失，而是换了位置。另一个实用细节是：客户端走新策略还是旧策略取决于 model id 与 provider 的路由分流，所以审静态 Prompt 文件并不足以判断运行时实际装了什么^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:229-242]。

### 规则归属的三问决策框架

文章把"规则放哪一层"收敛为三个问题：是否需要模型结合现场判断（"最小修改"留给 Prompt，`pending | in_progress | completed` 这类确定约束写进 Schema）；失效后损失能否恢复（注释多写一段可 Review 回滚，重复扣款、误删生产数据必须落到代码不变量、最小权限与人工审批）；哪一层离事实最近且能留下验证证据（文件存在性归文件系统，重复记账归数据库约束，规则有效性归回归评测与日志）^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:171-183]。样本中的具体机制正好是这套框架的注脚：`mounted read-only` 让"不要修改"从提醒变成系统强制，即使模型判断失误文件系统仍会拒绝写入^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:143-151]；记忆规则 "did the user say this?" 则要求写入前先过来源关——推断不能自动升级为用户事实，需带来源、时间、证据类型和修改入口^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:131-137]。

### 从样本结构反推上下文装配顺序

把三份样本叠起来可以归纳出一条装配层次：底部是稳定的产品身份、安全与行为原则；往上是搜索、引用、记忆、Artifact 等产品能力层；再往上是用户偏好、历史记忆、当前文件等会话状态；工具 Schema、Skills 注册表、网络白名单与文件系统配置则跟随具体环境进入请求^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:165-167]。这是工程归纳而非 Anthropic 官方架构，但它解释了为何"系统提示词"这个概念正在失效——人们看到的文件是多层注入的合成产物，作者更倾向于用"规则控制面"取代单一大 Prompt 的心智模型，与 [[entities/agent-harness-12-components-7-decisions|Agent Harness 十二组件]] 的分层视角互相印证。清理存量 Prompt 时，作者建议每条规则登记六项：来源、使用频率、失效代价、当前归属、验证方式、回滚入口，再按"每次都要用的原则留 Prompt / 接口语义收回 Schema / 专业做法迁 Skill / 长期事实进带来源的记忆 / 不可逆动作交权限审批 / 每次改动绑定回归与日志"六条去向重新安置^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:330-341]。

## 实践启示

1. 抵制"出问题就往系统提示词补一句"的肌肉记忆：每条新规则先问它解决什么失败、什么条件下触发、失效代价多大，再决定放 Prompt、Schema、Skill、记忆还是权限层^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:171-183]。
2. 涉及付款、删除、外发、合并 PR 等不可逆动作的规则，永远不要只写在 Prompt 里——用最小权限、只读挂载、独立审批和执行层拒绝来兜底，Prompt 只负责解释这些边界存在的原因^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:302-312]。
3. 工具接口要替 Prompt 承担确定性约束：把合法状态枚举、参数、返回结构和细粒度错误语义（断言失败 vs 依赖缺失 vs 超时 vs 权限拒绝）写进 Schema，否则模型会把环境问题误判成代码问题并反复改错方向^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:271-277]。
4. 记忆写入前先过来源测试（"这是用户说的吗？"）：每条长期事实带来源、时间、证据类型和修改/删除入口，模型推断与用户陈述分层存放，防止错误记忆在多轮任务中滚雪球^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:292-300]。
5. 评测不能只看"测试通过"：固定重复事件、并发回调、回滚重试、乱序到达等回归场景，完整记录模型与 Harness 版本、加载的规则、权限拒绝和停止原因，使每次删规则/换模型都有前后对照与回滚入口^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:314-322]。
6. 清理既有 Prompt 时不要照抄 80% 的比例：挑一条高频任务链，用六项清单（来源/频率/失效代价/归属/验证/回滚）逐条盘点，说不清去向的规则先保留并观察它在真实轨迹中是否被用到^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md:326-341]。

## 与 Wiki 现有知识的关联

- 规则归属决策与 [[entities/system-prompt-vs-post-training-behavioral-constraints-2026|System Prompt vs Post-Training 行为约束]] 是同一问题的两面：前者是"放哪一层"，后者是"Prompt 与权重约束哪个更可靠"
- 与 [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]] 一致：规则应下沉到 harness 的工具/记忆/权限层
- Prompt 膨胀与失效问题见 [[entities/attention-collapse-context-management|Attention Collapse 与上下文管理]]

→ [[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定|原文存档]]
