---
title: "Claude code创始人最新揭秘：Claude Tag如何在团队内部「大杀四方」"
created: 2026-07-05
updated: 2026-11-14
type: entity
tags: [claude-code, claude-tag, coding-agent]
sources: [raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Claude code创始人最新揭秘：Claude Tag如何在团队内部「大杀四方」

Claude Code 创始人揭秘 Claude Tag 功能在团队协作中的实战应用，如何通过 Tag 系统实现团队级别的 AI 代码生成管理。 ^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]

## 核心内容

↑阅读之前记得关注+星标⭐️，😄，每天才能第一时间接收到更新^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


Claude Fable 5现在已经接入Claude Tag。CC之父Boris Cherny和CC产品经理Cat Wu最新对谈详细分享了从Claude Code到Claude Tag的演进之路，以及这项技术如何从工程团队普及到整个公司。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


对于Anthropic的员工来说，他们已经很难想象一个没有Claude参与的频道会是什么样。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


Claude Tag  被Andrej karpathy称为一个LLM交互范式变革，可以看我之前的文章：^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


[ Anthropic发布重大更新推出CC进化版Claude Tag，Karpathy：LLM第三次交互革命来了 ](<https://mp.weixin.qq.com/s?__biz=Mzg3MTkxMjYzOA==&mid=2247515896&idx=1&sn=409c75320b83ce0ea18f13b9e4b2c496&scene=21#wechat_redirect>)^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


这里我们看看Boris Cherny和Cat Wu是怎么解读的。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


对谈视频我附在文后了。

###  三级跳:从补全一行代码,到AI自己给自己排期

三年前,Anthropic才刚开始把AI引入日常工作流程。两年前,员工用的还是代码自动补全功能,AI能做的只是把正在写的这一行代码续写完整,决定写什么的仍然是人。后来Claude Code上线,带来了智能体,AI开始能够写出完整的函数、完整的文件,甚至完整的功能。而现在有了Claude Tag,AI可以独立完成一整个任务,写功能、跑实验、做数据分析,全流程都能自己搞定。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


这个转变被总结成三个阶段。第一阶段是一个人盯着屏幕,一行一行敲代码。第二阶段是一个人同时开着十个Claude,并行写多个功能。第三阶段就是现在,Claude开始主动驱动工作,参与的也不再只是一个人,而是一整个团队。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


###  什么是Claude Tag

过去用Claude,流程是打开对话框,提出问题,Claude完成任务后返回结果,工具和电脑操作也都由它代劳,但整个过程始终是被动响应。Claude Tag的不同之处在于主动性,它知道什么时候该介入,即便一个任务需要跑上几天甚至几周,它也会持续跟进,并且会记住之前交代过的事情。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


以前每次都要重新提醒Claude该做什么,现在只需要把它拉进一个工作群,它就会主动介入、完成工作。因为是在群聊里运行,所有人都能看到过程、都能参与进来,而且它会把这次交流记下来,下次直接沿用。^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


###  技术支撑:更长的自主运行时间,加上真正好用的记忆

支撑Claude Tag的,是Anthropic这些年在模型长时间自主运行能力上的投入。据介绍,按照最新的METR评测,现在的模型能够连续工作16个小时,而且已经很难准确判断它^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]


^[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方.md]

---

→ [[raw/articles/claude-code创始人最新揭秘claude-tag如何在团队内部大杀四方|原文存档]]

---
## 深度分析

### 从"被动响应"到"主动介入"：交互范式的关键跃迁

Boris Cherny 与 Cat Wu 在对谈中反复强调，Claude Tag 与以往任何 Claude 使用方式的最大差异不在模型能力，而在交互姿态。此前的对话框模式是典型的被动响应：用户打开窗口、提出请求、等待结果，Claude 在整场交互中始终处于被召唤的位置。Claude Tag 则把 Claude 变成群聊中的一个常驻成员——被拉进频道只是起点，此后它会自己判断何时介入、何时沉默，并在长达数天甚至数周的任务上持续跟进。更关键的是"记住"：它记得之前交代过的规则和偏好，下次直接沿用，省去了每次重新对齐语境的成本。Karpathy 将这类变化称为 LLM 交互范式的变革，其分量不亚于 Claude Code 带来的智能体化：交互重心从"人发起、AI 执行"转向"人设定目标与边界，AI 自主排期并推进"。对谈中总结的三级跳——从逐行补全，到独立写出完整函数与功能，再到 AI 自己给自己排期、驱动整个团队的工作节奏——正是这一范式转变的浓缩表述。

### 长时程自主与记忆：Claude Tag 的技术底座

支撑这种主动性的两项核心技术是长时间自主运行和可靠的记忆。按 METR 最新评测，模型已能连续工作约 16 小时，且已很难准确判断它到底还能再撑多久；而 Claude Tag 允许 Claude 给自己安排后续工作，把一个原本 16 小时的任务拆解、延展到几天、几周甚至几个月的时间尺度上推进。对谈中提到一个跑了近一个月的会话：它本质上是一场长期实验，Claude 每天检查数据、遇到问题直接提交修复，使用者只需查看新提交的 PR 和数据反馈。记忆能力则打磨得更久——从 Claude Code 阶段就一直在尝试，直到近期才"真正做对"。Claude 不仅能给自己设提醒，还能记住不同用户在不同时间给过的全部指令；例如告诉它只关注某一类问题、不要管其他类别，它会一直遵守这条规则，除非有人明确要求扩大关注范围。此外被反复提及的是"分寸感"：Claude 经过训练能判断何时该出面、何时保持安静，用户只需说一句"调整介入频率"，它同样会记住并照做。

### 公开频道中的最佳实践扩散与多人协同

Claude Tag 运行在公开频道里，这一设计选择带来了超出预期的组织效应：所有人都能看到资深用户如何使用它，再把用法搬到自己负责的项目上。对谈双方指出，无论公司内部还是客户一侧，这种最佳实践扩散在以往 AI 工具中并不常见，且速度出乎意料——从少数人使用到身边人跟进几乎无需推广。多人协同是另一层变化：过去是一个人单独与 Claude 交流，再把结果复制粘贴给团队；现在多个成员可以共同引导同一场会话。由此形成三重学习回路——人通过观察别人如何使用 Claude Tag 学会更好地使用它；Claude Tag 通过听取不同人给出的指导学会更好地配合团队；最终客户得到的结果融合了整个团队打磨一个 PR 的多方判断。这同时降低了参与代码的门槛：许多此前不敢改代码的人，无需面对终端、Git 和代码检出这些操作，也能直接介入。

### 从 Claude Code 到 Claude Tag 的任务迁移曲线

内部使用习惯呈现清晰的迁移路径：最初几乎所有任务都用 Claude Code 处理，最先交给 Claude Tag 的是按钮位置差几个像素、简单数据问题这类轻量修复；随着对产品越来越熟悉，交给它的任务复杂度持续上升——因为它能自己验证工作结果，运行在与移动端、桌面端共用的远程沙盒环境里，用的是同一套智能体 SDK，智能程度一致。有的频道里，Claude Tag 修好一个 bug 后会直接在群里回传验证视频，使用者甚至不需要离开对话界面去确认。最有说服力的数字是：产品团队中由 Claude Tag 提交的 PR 占比约 65%，且仍在持续上升；有产品经理同时用它在五到十个功能上做状态跟踪，每天给出一份汇总报告。与 Claude Code 相比的关键差异在交互节奏——交代一件事之后可以直接去做下一件事，不需要一直守着它、随时确认进度。下一步它计划从 Slack 扩展到 Microsoft Teams 等更多协作平台，让每位知识工作者都能用上。

## 实践启示

1. 把 AI 从"对话框"搬进"工作群"：与其每次重新提醒，不如给 Claude 一个常驻位置和明确的职责边界，让它主动介入并长期跟进。
2. 用"一次交代、长期有效"的方式积累规则：凡是希望 AI 长期遵守的偏好（关注范围、汇报方式、验证标准）都明确说一次让它记住，之后只在需要时调整。
3. 公开运行、共享观察：把 AI 会话放在团队可见的频道里，让最佳实践自然扩散，也让多人能共同引导同一场会话、融合多方判断。
4. 从小任务开始建立信任，再逐步加码：先交出低风险的小修复，验证 AI 的自我验证能力（回传视频、提交 PR）后再迁移更复杂的任务。
5. 用长时程任务重新定义 AI 的角色：允许 AI 把大任务拆解为跨天、跨周的推进计划，人只保留查看进展与反馈的环节。
6. 把"介入频率"当作明确的配置项管理：让 AI 学会何时该说话、何时保持安静，是团队接受度的关键，方向不对时直接告诉它调整即可。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

