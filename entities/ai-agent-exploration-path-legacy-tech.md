---
title: "十年老技术开发的 AI Agent 探索之路"
created: 2026-05-16
updated: 2026-09-05
description: Auto-generated placeholder
review_value: 5
sources: [raw/articles/ai-agent-exploration-path-legacy-tech]
review_confidence: 10
source: "[[raw/articles/ai-agent-exploration-path-legacy-tech|原文存档]]"
type: entity
value: 7
tags: [agent, ai]
---

# 十年老技术开发的 AI Agent 探索之路
作者：zhiyuanfu    ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> 曾经前端被戏称为"娱乐圈"——工具、框架层出不穷，今年🔥 的明年就过时。现在 AI 把这个周期压缩到了以月计：这个月的新概念，下个月可能就是旧闻。这篇文章，就是一个在"AI 娱乐圈"摸爬滚打的老开发，试图从月抛式的焦虑中找到不会过期的东西，为大家抛砖引玉。 

  * 4-6 个终端的并发上限，怎么突破 
  * 80% 的 AI 需求，10 行 Bash 就够了 
  * Vibe Coding 翻车全记录 
  * 24h 无人值守的代码开发 Agent 怎么造 
  * 从 Task-Driven 到 Goal-Driven 的认知跃迁 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md] 

###  第一章：起点——人是瓶颈 
此刻我的屏幕上开着 5 个终端。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
左上角，codex 正在跑一组单元测试，终端里绿色的 pass 和偶尔的红色 fail 交替滚动。右上角，gemini-cli 在按照我刚给的方案改一个接口的入参校验。左下角，claude 在根据最新的 API 变更生成文档。右边整块屏幕留给了 Cursor，里面同时开着两个 Agent 窗口——一个在重构组件，一个在补集成测试。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
看起来很酷？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
真实体验是这样的：codex 那个终端跑了五分钟没动静，我得翻上去看是卡住了还是在等确认；gemini 改完接口了，但我忘了它改的是哪个分支；claude 写的文档引用了一个旧接口名，因为我忘了告诉它刚才 gemini 改过了；Cursor 里的重构窗口弹了个确认框，我一直没注意到，白白等了十分钟。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这种模式的上限大概就是 4-6 个并发。再多，人脑的 context switch 就开始崩溃。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
人工并发有三个硬伤： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
限制  |  具体表现  |  后果 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
---|---|--- ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
吞吐有限  |  一天能管 4-6 个 Agent 窗口  |  任务量有硬上限 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
状态不稳定  |  上下文丢失、判断漂移、质量波动  |  上午管 5 个，下午犯困管不了 3 个 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
难以规模化  |  做成一次不难，稳定重复难  |  今天的成功经验明天就忘了 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
所以真正的命题不是"怎么让 AI 更聪明"。Agent 的价值不是替人做事，是把依赖人的高频工作，改造成可以持续执行、可观测、可复盘、可优化的系统。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** 人是瓶颈。但解决瓶颈的方式不是让 AI 替代人，而是让系统不再依赖人的实时在场。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
想明白这件事之后，我开始动手。但在造系统之前，我先学到了一条最重要的原则。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> ** 工程建议：  ** 如果你现在也在手动管多个 AI 终端，先别急着造系统。先记录一周：哪些操作是重复的？哪些切换是可以消除的？瓶颈清单比技术方案更重要。 
_ 你现在同时开几个 AI 窗口？上限是多少？评论区聊聊。  _ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

###  第二章：80% 的 AI 需求不需要 AI 
我开始认真折腾 AI 的时候，第一件事不是去调模型、搞 RAG，而是写了一套 Bash 脚本来自动化日常工作流。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
结果发现——  ** 80% 的"AI 需求"，根本不需要 AI。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
自动拉取代码跑测试？Bash。定时检查服务健康状态？  ` cron + curl  ` 。把 JSON 日志格式化成报表？  ` jq + awk  ` 。文件变更触发构建？  ` inotifywait + shell  ` 。这些东西不需要任何模型，10 行脚本就搞定了。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

    #!/bin/bash ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    # 例：定时健康检查 + 告警，不需要任何 AI
    while true; do ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
     status=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
     if [ "$status" != "200" ]; then ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
       curl -X POST "$WEBHOOK_URL" \ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
         -d "{\"msg\": \"API health check failed: HTTP $status\"}" ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
     fi ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
     sleep 300 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    done ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这让我提炼出后来最重要的一条原则： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> 代码优先于 Prompt。能用 10 行 Bash 解决的，别折腾 AI。 
听起来像废话？但你去看看市面上多少项目，明明一个  ` cron + curl  ` 就能搞定的定时数据采集，非要套一层 LangChain，加个 Agent 循环，搞个 tool calling，最后效果还不如写死的脚本稳定。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这个认知后来演化成了一个决策层级： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** 目标 → 代码 → CLI → Prompt → Agent。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
层级  |  适用场景  |  示例  |  不确定性 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
---|---|---|--- ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
目标层  |  想清楚到底要解决什么  |  想清楚后发现不需要写代码  |  最低 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
代码层  |  确定性逻辑  |  if/else、正则、模板引擎  |  低 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
CLI 层  |  组合现有工具  |  ` grep + jq + curl  ` 串流程  |  中低 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
Prompt 层  |  需要语义理解和判断  |  需求翻译、文案生成  |  中高 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
Agent 层  |  多步推理、动态决策、循环执行  |  自动修 bug、端到端流程  |  最高 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
每往上一层，不确定性增加一个量级，成本也增加一个量级。原则很简单：  ** 能在下层解决的，绝不上推。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** 能用 10 行 Bash 解决的，别折腾 AI。这不是反 AI，是尊重工程。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> ** 工程建议：  ** 拿到一个新需求时，从表格最底行往上看——先问"10 行 Bash 能搞定吗？"，再问"一次 LLM 调用够吗？"，最后才考虑 Agent。这个习惯会帮你省掉 80% 的过度工程。 
_ 你团队里有没有"明明脚本就能搞定，偏要上 AI"的项目？说出来让大家乐乐。  _ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

###  第三章：Vibe Coding 翻车记 
知道了"什么时候该用 AI"，接下来就是动手造系统了。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
但在造出正经系统之前，我先翻了一次车。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
24h 打工人项目初期，我也尝试过 Vibe Coding：不写 spec、不做设计，直接跟 AI 说"帮我做个 XXX"，然后看着它一顿操作猛如虎。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
下面是真实时间线： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    Day 1-3   ✨ "wow 这 AI 真厉害" ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              几句话出一个完整页面，说需求就能跑通流程 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              产出速度惊人，成就感爆棚 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    Day 7     ⚠️  代码开始乱了 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              AI 对功能的实现越来越差 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              陷入"打地鼠"——修了这个 bug 冒出那个 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              告诉它"这里有问题"，它改了之后引入两个新问题 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    Day 14    🔥 被迫亲自打开每个文件浏览 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              大量过度设计、冗余逻辑 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              三层抽象解决一个本该一个函数搞定的问题 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              重复的工具方法散落在五六个文件里 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    Day 15    🔧 整整一天"设计与实现对齐" ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              把 AI 写的代码和手写的设计文档一一对照 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              逐个重构，这一天比前两周加起来都累 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
              但这一天的价值，也比前两周加起来都大 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
Vibe Coding 的问题本质很简单：它是"先易后难"。前期省掉的设计时间，后期会以 10 倍的 debug 时间还回来。代码越写越多，AI 的上下文越来越混乱，每一次修改都在给系统埋雷。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
SDD 恰好相反。写 spec 很慢，做设计很枯燥，但一旦 spec 写清楚了，后面的执行、验证、迭代全都有据可循。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
大路平坦宽阔，但人偏偏喜欢走捷径。Vibe Coding 就是那条看起来省事的小路——走着走着就发现，路越来越窄，荆棘越来越多，最后还得退回来走大路。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** Vibe Coding 是先易后难。SDD 是先难后易。大道如夷，而民好径。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
Day 15 那一天的"设计与实现对齐"很痛苦。但正是这一天，建立了让系统后续能自动运转的全部基础——设计文档、架构约束、SDD 流程。没有这一天，就没有后面的 24h 打工人。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> ** 工程建议：  ** 如果你现在正在 Vibe Coding，享受前几天的快感没问题，但第三天就要开始补 spec。越早补，代价越小。哪怕只有三段话——要做什么、不做什么、怎么算完成。 
_ 你 Vibe Coding 翻车过吗？最后是怎么收场的？  _ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

###  第四章：24h 打工人——第一个真正的系统 
翻车之后，我重新来过。这一次，先设计再动手。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
场景是这样的：用户提了个 bug——"搜索结果列表的分页有问题，切换页码后数据没更新"。半小时后，AI 自动完成了分析需求、生成技术方案、拆解任务、并发执行前后端代码修改、通知我 review。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
不是 demo。不是手动跑了五遍调通的演示。是一个真正能 24 小时无人值守运行的系统。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
我叫它  ** 24h 打工人  ** 。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

####  为什么选 CLI 而不是 API 
先说一个很多人会问的问题。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
答案不是教条，是阶段性选择。在我当时的场景里，codex、gemini-cli、claude-code 这些工具已经内置了读文件、改代码、跑命令的能力。它们本身就是完整的 Agent——有上下文管理、有工具调用、有错误处理。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
我要做的不是重新造一个 Agent，而是造一个"管理 Agent 的调度层"。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> 在我当时的阶段里，CLI 是最低成本、最容易 debug、最利于 AI 直接读取和修复的方案。 
这不是在论证"CLI 一定优于 API"。等哪天我需要更细粒度的控制、更低的延迟、更高的并发，会毫不犹豫切到 API。工具是手段，不是信仰。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

####  自建调度层：核心架构 
核心架构四个字就能概括：  ** 文件 + 轮询  ** 。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

    # 调度伪代码
    class AgentScheduler: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        def __init__(self): ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
            self.queue = FileQueue("storage/feedbacks/") ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
            self.tools = ToolProber(["codex", "gemini-cli", "claude"]) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
            self.state = JsonStateManager("storage/state/") ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        def run(self): ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
            while True: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                task = self.queue.poll() ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                if not task: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    time.sleep(10) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    continue ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                tool = self.tools.get_available()  # 自动选可用工具 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                try: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    result = tool.execute(task.prompt, task.workspace) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    self.state.update(task.id, status="done", output=result) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                except QuotaExhausted: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    self.tools.cooldown(tool, duration=300)  # 5 分钟冷却 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    self.queue.requeue(task)  # 重新入队，换工具执行 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                except Exception as e: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
                    self.state.update(task.id, status="failed", error=str(e)) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
调度层做四件事： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 1. ** 接收任务  ** ：用户反馈进来，写入文件队列 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 2. ** 分发执行  ** ：轮询队列，调用 CLI 执行 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 3. ** 状态管理  ** ：记录每一步的输入输出，持久化到文件 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 4. ** 失败切换  ** ：某个 CLI 配额用完，自动换下一个 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
选型极其简单：  ** 文件系统存储 + 轮询调度 + JSON 状态管理。  ** 不是在论证"文件系统一定优于数据库"。对一个还在高速迭代的 Agent 系统来说，文件系统的好处很实际：出问题直接让 AI 看文件，不用查数据库；方便 Git 版本控制；本地和生产环境更一致。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

####  SDD：让 AI 的每一步都有据可查 
这套系统里最核心的概念是  ** SDD（Spec-Driven Development）  ** 。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
很多人把 SDD 理解为"先写文档再开发"。但在 Agent 场景里，SDD 更重要的作用是：  ** 把一件事从模糊想法逐步转成可执行单元，并且把这个过程完整留下来。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
每个需求处理完会留下一组文档： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    storage/feedbacks/2026-01-15/20260115-143021-a1b2c3/ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    ├── sdd/ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    │   ├── spec.md      # 规格说明：目标、验收标准 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    │   ├── plan.md      # 技术方案：涉及文件、实现步骤 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    │   └── tasks.md     # 任务清单：每个任务的描述和状态 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    ├── tasks.json       # 任务执行状态（供程序读取） ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    └── debug/ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        ├── prompts/     # 每一步的 prompt ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        └── agent.log    # 执行日志 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
` spec.md  ` 把"分页有问题"变成"切换页码后列表数据未刷新，原因是查询参数未传递 page 参数"。  ` plan.md  ` 把问题变成方案。  ` tasks.md  ` 把方案拆成可执行的步骤，每一步都有明确的输入、输出和完成标准。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
如果一次执行没有留下足够上下文，你就回答不了四个关键问题： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

  * 它当时看到了什么输入？
  * 为什么做出这个判断？
  * Prompt 在哪个环节失效了？
  * 哪些动作已经足够稳定，可以固化成 Skill？
没有这些记录，系统就只能不断"重来一次"；有了这些记录，系统才可能真正"学会下一次做得更好"。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** 留痕不是为了 debug，而是为了进化。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

####  智能并发策略 
任务拆解完成后，系统按项目分组执行： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
策略  |  具体做法  |  理由 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
---|---|--- ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
组间并发  |  前端任务和后端任务同时跑  |  代码在不同目录，不会冲突 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
组内串行  |  同一个前端项目的任务排队执行  |  可能修改同一文件，避免冲突 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
失败隔离  |  单个任务失败不影响其他组  |  故障不扩散 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
并行的本质不是"同时做很多事"，而是"让每件事都不需要等别人"。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

####  工具失败自动切换 
AI CLI 工具经常遇到配额限制。我的方案是配合 Tool Prober 定时探测工具可用性： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    正常流程：task → codex（可用）→ 执行成功 → 完成 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    失败切换：task → codex（配额耗尽）→ gemini-cli → 执行成功 → 完成 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    全部不可用：task → 等待 → 5 分钟后自动探活 → 恢复后继续 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
单个工具挂了不影响整体，配额耗尽自动切换。这套机制让系统真正做到了 24 小时无人值守——从 4 个终端的手忙脚乱，到 20-30 个并发任务的稳定执行。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> ** 工程建议：  ** 起步阶段，文件系统 + JSON 状态比数据库更适合 Agent 系统。原因很实际——出了 bug 可以直接让 AI 读文件排查，不需要教它查数据库。等系统稳定到需要事务和并发锁的时候，再升级不迟。 
_ 你的 Agent 系统出了 bug，排查过程是什么样的？靠翻日志还是靠猜？  _ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
_ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

###  第五章：Agent 自己修了自己的 bug 
花了一整天做"设计与实现对齐"之后不久，一个有意思的事情发生了。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
那天我在用 24h 打工人的需求澄清页面，发现了一个 bug：无法选择待确认问题的选项，也没有提交按钮。页面渲染出来了，但交互完全不能用。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
以前遇到这种事，流程是：打开代码 → 定位问题 → 手动修复 → 测试 → 提交。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
但这次我换了个做法——直接通过 24h 打工人自己的反馈系统提交了这个 bug。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    [用户反馈] 需求澄清页面有 bug：无法选择待确认问题的选项，也没有提交按钮。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    ↓ 系统自动触发 SDD 流程 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    [spec.md] 目标：修复需求澄清页面的交互组件渲染问题 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
             验收标准：选项可点选，提交按钮可见且可用 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    ↓ AI 生成方案并拆解任务 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    [plan.md] 定位：RadioGroup 组件未正确绑定 onChange 事件 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
             方案：修复组件 props 传递，补充提交按钮渲染逻辑 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    ↓ 我确认澄清结果 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    [执行] 分析代码 → 定位问题 → 修改组件 → 验证修复 → 完成 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    ↓ 企微通知：任务已完成 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** 它自己修复了自己的 bug。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
整个过程我只做了两件事：提交反馈、确认澄清。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这件事背后有一个严肃的前提：  ** 自举不是凭空发生的。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
还记得 Day 15 花了整整一天做的"设计与实现对齐"吗？那一天的工作产出不只是修复了代码，更重要的是建立了三样东西： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 1. ** 清晰的设计文档  ** —— AI 知道每个模块该做什么、不该做什么。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 2. ** SDD 流程  ** —— spec → plan → tasks 的标准路径，AI 按照同样的方式处理所有需求，包括关于自身的需求。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
 3. ** constitution.md  ** —— 架构约束文件，定义了代码组织规范、命名规则、模块边界。AI 在生成方案时会自动遵循这些约束。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
没有这些基础设施，AI"自己修自己"就只是碰运气。有了这些，它才能在框架内工作，而不是自由发挥。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
从 Vibe Coding 的混乱，到一天的阵痛，到 Agent 自举——这条路的因果链非常清楚。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
** 捷径的尽头是弯路，大道的尽头是自由。  ** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
> ** 工程建议：  ** 自举的前提是 constitution.md（架构约束文件）。不需要写得多长，但至少要覆盖三件事：目录结构约定、模块边界、命名规则。有了它，AI 才能在"框架内工作"而不是"自由发挥"。 
_ 你见过最惊艳的 AI 自举案例是什么？  _ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

## 深度分析
### 从方法论演进看 AI Agent 开发的认知阶段
这篇文章实际上勾勒了一条清晰的 AI Agent 开发认知进化路径，呈现四个递进层次： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**第一层：人肉并发阶段的瓶颈意识** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
作者在第一章揭示的核心矛盾，不是"AI 能力不足"，而是"人的注意力有限"。4-6 个终端的并发上限并非技术限制，而是人脑的 context switch 瓶颈。这个认知非常关键——它把问题定义从"如何让 AI 更强"转变为"如何让系统脱离人的实时在场"。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这对应着 Task-Driven 阶段的早期：人仍然是调度核心，但已经意识到手动调度的天花板。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**第二层：技术选代的成本意识** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
第二章的决策层级（目标 → 代码 → CLI → Prompt → Agent）本质上是作者经过大量实践后的经验沉淀。核心洞察是：**每上升一层，不确定性增加一个量级，成本也增加一个量级**。这解释了为什么"脚本能搞定的偏要上 LangChain"会成为反模式——这不是技术能力问题，而是工程判断力问题。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这个层级模型对 Agent 开发者的实际指导意义在于：它提供了一个清晰的技术选型框架，避免过度工程。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**第三层：系统化建设的工程化转向** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
第三章的 Vibe Coding 翻车和第四章的 SDD 实践，构成了一组对照实验。Vibe Coding 的失败证明了"先易后难"的债务积累模式——前期省掉的设计时间会以 10 倍的 debug 时间还回来。而 SDD 的价值在于：把模糊的需求变成清晰的路径，并把整个过程完整记录下来。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
这里的关键工程原则是：**spec 不是文档，而是 Agent 与人之间的契约**。没有契约，Agent 只能在混乱中自由发挥。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**第四层：自举与 Goal-Driven 的跃迁** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
第五章的"Agent 自己修自己的 bug"是文章的方法论高点。自举不是偶然发生的奇迹，而是 SDD + constitution.md + 调度层共同构成的基础设施成熟后的自然产物。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
第六章的"脚手架 > 模型"则是这个认知的自然延伸：投入回报对比显示，脚手架升级（成本 +50%）带来的效果提升（+200%）远高于模型升级（成本 +300%，效果 +20%）。这颠覆了"最贵模型 = 最好效果"的直觉。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

### 协议层演进的产业信号
第七章梳理的协议层进展（Responses API、MCP、A2A）指向一个更大的趋势：**Agent 开发正在从"框架之争"走向"协议 + runtime + control plane 之争"**。这个判断的战略含义是：对个人开发者而言，理解协议背后的设计理念比追框架 API 更有长期价值。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

### Goal-Driven 的本质
第八章提出的 Task-Driven vs Goal-Driven 范式转变，核心洞见是：**Task-Driven 解决的是执行问题，Goal-Driven 解决的是迭代问题**。前者让系统开始能跑，后者才让系统开始能持续向前。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
Goal-Driven 的五个前提（目标清晰、边界清晰、状态可见、过程留痕、权限可控）实际上是对 Task-Driven 基础设施的全面升级——它要求系统不仅能稳定执行，还要能自主判断方向。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

### 这篇文章的局限与补充
值得注意的的是，本文是一个资深开发者的个人经验总结，有些判断依赖于特定的阶段和场景： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
1. **CLI 选型的阶段性**：作者选择 CLI 是因为当时的工具链已经足够成熟。但随着 Responses API 等 runtime 的收敛，未来 API 化可能是主流。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
2. **文件系统存储的适用性**：作者坦承文件系统是"高速迭代阶段"的合理选择，但对于需要事务、并发锁的生产系统，最终仍需升级到数据库。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
3. **脚手架 > 模型的结论是情境依赖的**：当模型能力成为瓶颈时（例如需要更复杂的推理），模型升级的优先级会上升。这不是一个绝对原则，而是资源分配的经验法则。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

## Related entities
- [[entities/十年老技术开发的-ai-agent-探索之路-v2|十年老技术开发的 AI Agent 探索之路]]
- [[entities/ai-agent-exploration-path-legacy-tech.md|十年老技术开发的 AI Agent 探索之路]]- [[entities/十年老技术开发的-ai-agent-探索之路|十年老技术开发的 AI Agent 探索之路]]- 十年老技术开发的 AI Agent 探索之路

## 实践启示
### 针对 Agent 开发者的具体建议
**优先级 checklist：先治理，后模型** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
如果你现在在构建 Agent 系统，按这个顺序投入： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
1. **目标表达**：能否清楚地说出"系统要完成什么、不完成什么"？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
2. **能力单元**：有哪些高频操作可以沉淀为 Skill（模板 + 规则 + 代码）？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
3. **运行时状态**：当前系统正在做什么，能否回答"它为什么失败"？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
4. **治理边界**：允许做什么，不允许做什么，权限如何控制？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
5. **评估反馈**：成功的标准是什么，如何持续校准？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**SDD 的最小实践** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
不需要一开始就建立完整的文档体系。从最小可用 SDD 开始： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

- `spec.md`：三段话——目标、不做什么、怎么算完成
- `plan.md`：涉及的文件列表 + 实现步骤
- `tasks.md`：可执行的任务单元，每步有明确输入输出
**constitution.md 的核心要素** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
至少覆盖三件事： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

- 目录结构约定（代码放哪里）
- 模块边界（什么归什么模块）
- 命名规则（统一的命名风格）
**工具失败切换的最小实现**^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
```python ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

# 伪代码：最简自动切换
tools = ["codex", "gemini-cli", "claude"]^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
for tool in tools: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    try: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        result = tool.execute(task) ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        return result ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    except QuotaExhausted: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        continue ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
    except Exception as e: ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        log.error(f"{tool} failed: {e}") ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
        continue ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
``` ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**避免 Vibe Coding 债务的早期预警** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
如果你是 Vibe Coding 用户，设置一个简单的检查点： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

- 第 3 天：开始补充 spec，哪怕只有几句话
- 第 7 天：检查代码质量——是否开始出现"打地鼠"现象？
- 第 14 天：评估上下文是否已经开始混乱？
**从 Task-Driven 到 Goal-Driven 的六步路径** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 步骤 | 做什么 | 核心产出 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
|---|---|---| ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 第一步 | 写清楚 spec | 要做什么、不做什么、怎么算完成 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 第二步 | 执行过程留痕 | Prompt 状态 输出 / 错误全记录 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 第三步 | 补 observability 和 eval | 知道为什么成功、为什么失败 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 第四步 | 高频动作沉淀为 Skill | 模板 + 规则 + 代码 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 第五步 | 引入调度和并发 | 调度层 + 轮询 + 失败切换 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
| 第六步 | 最后才尝试 Goal-Driven | 目标表达 + 治理边界 + 共享状态 | ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
**技术选型的协议意识** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
选技术栈时，优先看它是否兼容 MCP / Responses API 这些正在收敛的协议。自己造的胶水层越多，未来迁移成本越高。协议层是长期资产，框架是短期工具。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

### 评估你的 Agent 系统现在在哪一步
问自己四个问题： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
1. 我的系统能否稳定重复执行同一个任务？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
2. 如果执行失败，我能否回答"它为什么失败"？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
3. 我能否让 AI 自己修复一个 bug，而不需要我手动介入？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
4. 我的系统是否能自主判断下一步该做什么？ ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
如果四个问题都是"不确定"或"不能"，系统还在 Task-Driven 早期阶段，不要急于跳到 Goal-Driven。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

### 一个日常习惯的养成
**拿到新需求时，从底往上问**：先问"10 行 Bash 能搞定吗？"再问"一次 LLM 调用够吗？"最后才考虑 Agent。这个习惯会帮你省掉 80% 的过度工程。^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
---^[raw/articles/ai-agent-exploration-path-legacy-tech.md]
→ [[raw/articles/ai-agent-exploration-path-legacy-tech.md|原文存档]]^[raw/articles/ai-agent-exploration-path-legacy-tech.md]