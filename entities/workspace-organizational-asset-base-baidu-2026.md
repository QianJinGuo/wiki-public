---
title: "Workspace：面向 Agent 的组织资产基座（百度实践）"
created: 2026-08-26
updated: 2026-10-02
type: entity
tags: [workspace, organizational-assets, knowledge-base, agent, harness, sdd, baidu, summarize, docs-engineering]
sources: [raw/articles/workspace-organizational-asset-base-baidu-2026]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Workspace：面向 Agent 的组织资产基座（百度实践）

> 百度Geek说（何雪源）第一方实践：个人提效 ≠ 组织提效。Workspace 是「面向 Agent 的组织资产基座」——解决组织资产用什么形式组织、怎么流动的问题，让 Agent 通过它了解业务、参与业务。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## 三个卡点与组织提效四件事
业务提效困境的三个卡点：①个人经验难以复制、②跨角色背景对齐成本高、③工具生态各自为政——都不是「模型不够强」造成的，换更强模型也不会消失。devflow（基于 SDD 理念的百度 Native 研发工作流，每阶段硬门禁）让个人产出涨了，但团队需求交付数据没改善——**纵向的短板决定交付周期，横向的断点决定经验能不能变成组织能力**。组织提效要解决四件事：资产怎么攒起来（低成本记经验）、攒起来的东西怎么流动、流程和角色职责变化、交付质量持续稳定。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## Workspace 是什么
不是文档库、不是 skill 库、也不是把几个仓库放一起。它是**面向 Agent 的组织资产基座**：把团队的知识库、代码库、iCafe 空间、skill 都放到一个代码库下，Agent 能访问所有业务资产还能自行验证产出。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

**关键原则：Workspace 不存知识的快照或备份，只存摘要和索引。** 不是把外部内容整理成文本存进去，而是构建外部资产的索引，让 Agent 知道查询某问题该去哪个平台获取——固有流程和方式不变（依旧在知识库写文档、iCafe 记录任务），Workspace 不直接改变工作模式，这也是它能推广的原因。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## 组织结构
README.md 是规则唯一源（AGENTS.md/CLAUDE.md 都是指向它的软链，三端共读，不把软链替换成副本）；`.claude/[本地运行时路径已隐藏]` 三端桥接只放软链（新增 agent/skill 只改一处，解决多 harness 生态）。docs 分两层按「内容会怎样失效」划分不按主题：**知识层**（docs/knowledge/）放概念定义/外部系统入口/带条件的经验判断——被新证据推翻时回头改旧页；**活动层**（docs/activity/）放每次迭代做了什么/怎么做的/验证结果——不失效只增不改。repos 用 git submodule 引入只读。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

四个关键文件：README（规则唯一源）、docs/README（路由表，只描述信息怎么找，外部平台是权威源 docs 只是索引层）、docs/INDEX（资产表一行一条 + 签名短哈希判断缓存落后）、docs/LOG（按日期倒序变更日志，留「为什么」依据）。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## summarize skill：资产流动闭环的最后一环
Workspace 能不能越来越好用，关键在 summarize skill（迭代收尾写回）——没有写回，Agent 每次都从零开始，Workspace 退化成静态文档库。三条设计取舍：**强制执行**（会话最后必须执行，写回不靠自觉）、**尽可能不打扰用户**（减少人的决策，写回变成需要人配合的手续就会被跳过）、**宁多记不漏记**（多余经验页只是噪声，丢掉的经验是下次重新踩一遍，拿不准写进不会失效的 activity 层）。分流表：客观事实/概念规则→knowledge、带条件判断/踩坑→experience-<主题>.md、外部系统入口→sources.md、做了什么怎么验证→activity/<日期>-<主题>.md、**本版参数阈值数值留 activity 不进 knowledge**（参数会随调整变、进知识层立刻过期还被当规则引用）。旧页被推翻时回头改原页（否则两份矛盾真相、检索命中随机）；同步 INDEX/LOG；补双向引用。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## 实践：RocketMQ Workspace（组织级）
核心产品 Workspace，文档组织三层，沉淀 20 个 skill 分六组覆盖需求讨论→开发验收→排障→封线→资产沉淀完整链路，Workflow 四阶段 Spec Driven Development。示例：Bug 修复（diagnosing-bugs 定位→to-icafe-card 建卡→spec-workflow 走方案设计/开发/验收/收尾四步→Patch+交付报告）；新功能开发（spec-workflow 走 spec driven 流程）。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## 深度分析

**1. 「个人提效 ≠ 组织提效」的传导断链是这次实践最有分量的实证。** devflow 深接厂内生态（iCafe/iCode/知识库/iAPI）加每阶段硬门禁，个人产出确实涨了——作者一个人 tmux 开 3-5 个会话并行干活；但从团队视角看，需求交付数据纹丝不动。原因不是哪一环做错了，而是传导链本身断了：纵向（单个需求周期内）短板卡在对齐成本，横向（跨需求跨人）断点卡在经验复制——403 报错作者三行规则解决，同事遇到同样的错还得重走一遍甚至误判为权限不足。这组数据给「超级兵叙事」提供了第一方反例：一个人变强，组织吞吐不变。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

**2. 「不存快照只存索引」本质是一个采纳成本决策，而非存储架构决策。** 常见的组织知识库失败路径是：要求大家把外部内容搬进新系统 → 搬运是额外劳动 → 本职工作一挤就被跳过 → 库变陈地。Workspace 反过来：文档还在原知识库写、任务还在 iCafe 记，Workspace 只建索引告诉 Agent「去哪查」。固有能力源不变、Agent 层新增索引，等于把迁移成本压到零，推广阻力自然消失。这是「让新系统长在旧习惯上」而不是「让旧习惯迁就新系统」的典型设计。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

**3. knowledge/activity 两层的划分轴是失效模式，不是主题分类。** 传统文档目录按主题（产品/技术/流程）组织，但这套设计按「内容会怎样失效」切分：概念定义和带条件的经验判断会过期——被新证据推翻时要回头改旧页，否则两份矛盾真相并存、检索命中随机；活动记录只增不改——每条都是当时的真实快照，永远有效。分流表的第五条是这条原则的极致体现：本版参数阈值数值留 activity 不进 knowledge，因为参数随下次调整就变，进知识层会立刻过期还被当成规则引用。用失效生命周期给信息建模，而不是用主题给信息归类，这是数据生命周期设计思路。参见 [[concepts/agent-memory-lifecycle-philosophies]]。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

**4. summarize 强制写回是让知识库不退化的唯一机制保障。** 作者判断的关键在一条铁律：没有写回，Agent 每次会话都从零开始，Workspace 退化成静态文档库。而写回一旦依赖人的自觉就会断环——人会忘、会嫌麻烦，闭环必死。三条取舍环环相扣：强制执行（会话最后必须跑，不靠自觉）、尽可能不打扰用户（写回变成需要人配合的手续就会被跳过）、宁多记不漏记（多余经验页只是噪声，丢掉的经验是下次重踩一遍，拿不准写进不会失效的 activity 层）。这印证了一个更普遍的判断：知识库的活性不取决于存储设计，而取决于写回通道是否被内建到工作流的协议里。参见 [[concepts/knowledge-base-output-flywheel]]。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

**5. 三端软链桥接用「单点定义 + 多点视图」化解多 harness 生态割裂。** 第三个卡点是工具生态各自为政：Claude 上攒的 skill 在 Comate/Codex 上用不了。解法不是造一个统一 harness，而是在仓库里放 `.claude/`、`[本地运行时路径已隐藏]`、`.comate/` 三套桥接目录，里面只放软链指向同一份 skills/agents——新增 agent 或 skill 只改一处，三端同时生效；README 同理，AGENTS.md/CLAUDE.md 都是指向它的软链，规则只维护一份。这和 docs/README 只做路由不复制外部内容是同一思想的两处落地：权威源唯一，其余全是视图。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## 实践启示

- **组织级 AI 提效先审计「经验传播断点」，再谈模型升级。** 遇到团队吞吐上不去，先区分是纵向短板（单需求内对齐慢）还是横向断点（经验不能复制）——devflow 的教训表明换更强模型两者都治不了。一个具体动作：让每个人把最近一次「自己搞定但同事大概率会踩」的坑写下来，数一下这些坑有多少已经第二次、第三次出现。断点多，先建基座再买算力。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]
- **知识库设计先问「这条内容会怎样失效」，再决定放哪层。** 会过期被推翻的（概念、带条件判断）放知识层并接受回头改旧页的维护义务；只增不改的（做了什么、怎么验证的）放活动层永不修改；会随配置漂移的数值（参数阈值）留在活动层当历史证据，绝不进知识层。两层都放不下的，停下来问人而不是硬塞。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]
- **写回机制必须是会话协议的一部分，而非依赖自觉。** 把 summarize 类技能设为会话收尾的强制步骤，设计上尽量少要人做决策；判断「记多了」还是「记漏了」时选前者——噪声可以治理，丢失的经验是真实的重复成本。同步维护索引和按日期倒序的日志，半年后还能查到某个约定当初为什么这么定。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]
- **多工具团队建基座时用「单点权威源 + 软链视图」降低维护面。** 规则和技能都只在一处维护，各 harness 通过软链共享；不要把软链替换成副本，副本一旦出现就会分叉。同理，docs 入口只做「信息怎么找」的路由，不复制外部平台内容——权威源在哪个平台就让它留在哪个平台。^[raw/articles/workspace-organizational-asset-base-baidu-2026.md]

## 相关
与 [[entities/ai-native-organization-methodology-ye-xiaochai-sdd-2026|AI-Native 组织方法论（SDD）]]、[[entities/sdd-practice-lattice-harness-team-ai-coding|Lattice Harness 团队 SDD 实践]]、[[entities/spec-as-aios-anti-entropy-architecture-gaode-ai-native-series-2|Spec as AIOS（AGENTS.md 类）]] 同属 spec-driven/组织工程主题；本文贡献是「面向 Agent 的组织资产基座」——不存快照只存索引 + knowledge/activity 两层 + summarize 写回闭环。与 [[entities/ai-true-moat-organizational-capability|组织能力护城河]]、[[entities/ai-true-moat-not-llm-but-organization|AI 时代护城河不是大模型]] 互补（组织资产 vs 组织能力）。→ [[raw/articles/workspace-organizational-asset-base-baidu-2026|原文存档]]
