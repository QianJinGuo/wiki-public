---
title: "十年老技术开发的 AI Agent 探索之路"
created: 2026-05-16
source: "[[raw/articles/十年老技术开发的-ai-agent-探索之路|原文存档]]"
type: entity
value: 8
tags: [claude-code, agent, ai]
review_value: 5
review_confidence: 7
  - raw/articles/十年老技术开发的-ai-agent-探索之路
updated: 2026-08-01
sources:
  - raw/articles/十年老技术开发的-ai-agent-探索之路
---

[[raw/articles/十年老技术开发的-ai-agent-探索之路]] ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

# 十年老技术开发的 AI Agent 探索之路

作者：zhiyuanfu

> 曾经前端被戏称为"娱乐圈"——工具、框架层出不穷，今年🔥 的明年就过时。现在 AI 把这个周期压缩到了以月计：这个月的新概念，下个月可能就是旧闻。这篇文章，就是一个在"AI 娱乐圈"摸爬滚打的老开发，试图从月抛式的焦虑中找到不会过期的东西，为大家抛砖引玉。

* 4-6 个终端的并发上限，怎么突破
* 80% 的 AI 需求，10 行 Bash 就够了
* Vibe Coding 翻车全记录
* 24h 无人值守的代码开发 Agent 怎么造
* 从 Task-Driven 到 Goal-Driven 的认知跃迁

###  第一章：起点——人是瓶颈
此刻我的屏幕上开着 5 个终端。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
左上角，codex 正在跑一组单元测试，终端里绿色的 pass 和偶尔的红色 fail 交替滚动。右上角，gemini-cli 在按照我刚给的方案改一个接口的入参校验。左下角，claude 在根据最新的 API 变更生成文档。右边整块屏幕留给了 Cursor，里面同时开着两个 Agent 窗口——一个在重构组件，一个在补集成测试。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
看起来很酷？^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
真实体验是这样的：codex 那个终端跑了五分钟没动静，我得翻上去看是卡住了还是在等确认；gemini 改完接口了，但我忘了它改的是哪个分支；claude 写的文档引用了一个旧接口名，因为我忘了告诉它刚才 gemini 改过了；Cursor 里的重构窗口弹了个确认框，我一直没注意到，白白等了十分钟。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
这种模式的上限大概就是 4-6 个并发。再多，人脑的 context switch 就开始崩溃。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
人工并发有三个硬伤：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
限制  |  具体表现  |  后果^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
---|---|---^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
吞吐有限  |  一天能管 4-6 个 Agent 窗口  |  任务量有硬上限^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
状态不稳定  |  上下文丢失、判断漂移、质量波动  |  上午管 5 个，下午犯困管不了 3 个^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
难以规模化  |  做成一次不难，稳定重复难  |  今天的成功经验明天就忘了^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
所以真正的命题不是"怎么让 AI 更聪明"。Agent 的价值不是替人做事，是把依赖人的高频工作，改造成可以持续执行、可观测、可复盘、可优化的系统。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 人是瓶颈。但解决瓶颈的方式不是让 AI 替代人，而是让系统不再依赖人的实时在场。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
想明白这件事之后，我开始动手。但在造系统之前，我先学到了一条最重要的原则。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** 如果你现在也在手动管多个 AI 终端，先别急着造系统。先记录一周：哪些操作是重复的？哪些切换是可以消除的？瓶颈清单比技术方案更重要。
_ 你现在同时开几个 AI 窗口？上限是多少？评论区聊聊。  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

_
_

###  第二章：80% 的 AI 需求不需要 AI
我开始认真折腾 AI 的时候，第一件事不是去调模型、搞 RAG，而是写了一套 Bash 脚本来自动化日常工作流。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
结果发现——  ** 80% 的"AI 需求"，根本不需要 AI。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

自动拉取代码跑测试？Bash。定时检查服务健康状态？  ` cron + curl  ` 。把 JSON 日志格式化成报表？  ` jq + awk  ` 。文件变更触发构建？  ` inotifywait + shell  ` 。这些东西不需要任何模型，10 行脚本就搞定了。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    #!/bin/bash
    # 例：定时健康检查 + 告警，不需要任何 AI
    while true; do
      status=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health) ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
      if [ "$status" != "200" ]; then^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

        curl -X POST "$WEBHOOK_URL" \^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         -d "{\"msg\": \"API health check failed: HTTP $status\"}" ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
      fi
      sleep 300
    done
这让我提炼出后来最重要的一条原则：
> 代码优先于 Prompt。能用 10 行 Bash 解决的，别折腾 AI。
听起来像废话？但你去看看市面上多少项目，明明一个  ` cron + curl  ` 就能搞定的定时数据采集，非要套一层 LangChain，加个 Agent 循环，搞个 tool calling，最后效果还不如写死的脚本稳定。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
这个认知后来演化成了一个决策层级：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 目标 → 代码 → CLI → Prompt → Agent。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
层级  |  适用场景  |  示例  |  不确定性^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
---|---|---|---^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
目标层  |  想清楚到底要解决什么  |  想清楚后发现不需要写代码  |  最低^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
代码层  |  确定性逻辑  |  if/else、正则、模板引擎  |  低^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
CLI 层  |  组合现有工具  |  ` grep + jq + curl  ` 串流程  |  中低^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
Prompt 层  |  需要语义理解和判断  |  需求翻译、文案生成  |  中高^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
Agent 层  |  多步推理、动态决策、循环执行  |  自动修 bug、端到端流程  |  最高^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
每往上一层，不确定性增加一个量级，成本也增加一个量级。原则很简单：  ** 能在下层解决的，绝不上推。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 能用 10 行 Bash 解决的，别折腾 AI。这不是反 AI，是尊重工程。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** 拿到一个新需求时，从表格最底行往上看——先问"10 行 Bash 能搞定吗？"，再问"一次 LLM 调用够吗？"，最后才考虑 Agent。这个习惯会帮你省掉 80% 的过度工程。
_ 你团队里有没有"明明脚本就能搞定，偏要上 AI"的项目？说出来让大家乐乐。  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

_
_

###  第三章：Vibe Coding 翻车记
知道了"什么时候该用 AI"，接下来就是动手造系统了。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

但在造出正经系统之前，我先翻了一次车。
24h 打工人项目初期，我也尝试过 Vibe Coding：不写 spec、不做设计，直接跟 AI 说"帮我做个 XXX"，然后看着它一顿操作猛如虎。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
下面是真实时间线：
    Day 1-3    ✨ "wow 这 AI 真厉害"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 几句话出一个完整页面，说需求就能跑通流程^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 产出速度惊人，成就感爆棚
    Day 7     ⚠️  代码开始乱了^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 AI 对功能的实现越来越差
                 陷入"打地鼠"——修了这个 bug 冒出那个^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 告诉它"这里有问题"，它改了之后引入两个新问题^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    Day 14    🔥 被迫亲自打开每个文件浏览^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 大量过度设计、冗余逻辑
                 三层抽象解决一个本该一个函数搞定的问题
                 重复的工具方法散落在五六个文件里
    Day 15    🔧 整整一天"设计与实现对齐"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 把 AI 写的代码和手写的设计文档一一对照^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                 逐个重构，这一天比前两周加起来都累
                 但这一天的价值，也比前两周加起来都大
Vibe Coding 的问题本质很简单：它是"先易后难"。前期省掉的设计时间，后期会以 10 倍的 debug 时间还回来。代码越写越多，AI 的上下文越来越混乱，每一次修改都在给系统埋雷。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
SDD 恰好相反。写 spec 很慢，做设计很枯燥，但一旦 spec 写清楚了，后面的执行、验证、迭代全都有据可循。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
大路平坦宽阔，但人偏偏喜欢走捷径。Vibe Coding 就是那条看起来省事的小路——走着走着就发现，路越来越窄，荆棘越来越多，最后还得退回来走大路。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** Vibe Coding 是先易后难。SDD 是先难后易。大道如夷，而民好径。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

Day 15 那一天的"设计与实现对齐"很痛苦。但正是这一天，建立了让系统后续能自动运转的全部基础——设计文档、架构约束、SDD 流程。没有这一天，就没有后面的 24h 打工人。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** 如果你现在正在 Vibe Coding，享受前几天的快感没问题，但第三天就要开始补 spec。越早补，代价越小。哪怕只有三段话——要做什么、不做什么、怎么算完成。
_ 你 Vibe Coding 翻车过吗？最后是怎么收场的？  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

_
_

###  第四章：24h 打工人——第一个真正的系统
翻车之后，我重新来过。这一次，先设计再动手。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

场景是这样的：用户提了个 bug——"搜索结果列表的分页有问题，切换页码后数据没更新"。半小时后，AI 自动完成了分析需求、生成技术方案、拆解任务、并发执行前后端代码修改、通知我 review。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
不是 demo。不是手动跑了五遍调通的演示。是一个真正能 24 小时无人值守运行的系统。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

我叫它  ** 24h 打工人  ** 。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  为什么选 CLI 而不是 API
先说一个很多人会问的问题。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
答案不是教条，是阶段性选择。在我当时的场景里，codex、gemini-cli、claude-code 这些工具已经内置了读文件、改代码、跑命令的能力。它们本身就是完整的 Agent——有上下文管理、有工具调用、有错误处理。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
我要做的不是重新造一个 Agent，而是造一个"管理 Agent 的调度层"。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> 在我当时的阶段里，CLI 是最低成本、最容易 debug、最利于 AI 直接读取和修复的方案。
这不是在论证"CLI 一定优于 API"。等哪天我需要更细粒度的控制、更低的延迟、更高的并发，会毫不犹豫切到 API。工具是手段，不是信仰。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  自建调度层：核心架构
核心架构四个字就能概括：  ** 文件 + 轮询  ** 。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


    # 调度伪代码
    class AgentScheduler:^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         def __init__(self):
            self.queue = FileQueue("storage/feedbacks/")^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

             self.tools = ToolProber(["codex", "gemini-cli", "claude"]) ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
             self.state = JsonStateManager("storage/state/")^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         def run(self):
            while True:
                task = self.queue.poll()^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                if not task:
                    time.sleep(10)
                    continue
                tool = self.tools.get_available()  # 自动选可用工具^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                try:
                    result = tool.execute(task.prompt, task.workspace)^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                    self.state.update(task.id, status="done", output=result) ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
                except QuotaExhausted:^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                    self.tools.cooldown(tool, duration=300)  # 5 分钟冷却^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                    self.queue.requeue(task)  # 重新入队，换工具执行^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                except Exception as e:^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

                    self.state.update(task.id, status="failed", error=str(e)) ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
调度层做四件事：
1. ** 接收任务  ** ：用户反馈进来，写入文件队列
2. ** 分发执行  ** ：轮询队列，调用 CLI 执行
3. ** 状态管理  ** ：记录每一步的输入输出，持久化到文件
4. ** 失败切换  ** ：某个 CLI 配额用完，自动换下一个
选型极其简单：  ** 文件系统存储 + 轮询调度 + JSON 状态管理。  ** 不是在论证"文件系统一定优于数据库"。对一个还在高速迭代的 Agent 系统来说，文件系统的好处很实际：出问题直接让 AI 看文件，不用查数据库；方便 Git 版本控制；本地和生产环境更一致。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  SDD：让 AI 的每一步都有据可查
这套系统里最核心的概念是  ** SDD（Spec-Driven Development）  ** 。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
很多人把 SDD 理解为"先写文档再开发"。但在 Agent 场景里，SDD 更重要的作用是：  ** 把一件事从模糊想法逐步转成可执行单元，并且把这个过程完整留下来。  ** ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
每个需求处理完会留下一组文档：
    storage/feedbacks/2026-01-15/20260115-143021-a1b2c3/ ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
    ├── sdd/
    │   ├── spec.md      # 规格说明：目标、验收标准^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    │   ├── plan.md      # 技术方案：涉及文件、实现步骤^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    │   └── tasks.md     # 任务清单：每个任务的描述和状态^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    ├── tasks.json       # 任务执行状态（供程序读取）^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    └── debug/
       ├── prompts/     # 每一步的 prompt^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

       └── agent.log    # 执行日志^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

` spec.md  ` 把"分页有问题"变成"切换页码后列表数据未刷新，原因是查询参数未传递 page 参数"。  ` plan.md  ` 把问题变成方案。  ` tasks.md  ` 把方案拆成可执行的步骤，每一步都有明确的输入、输出和完成标准。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
如果一次执行没有留下足够上下文，你就回答不了四个关键问题：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


* 它当时看到了什么输入？
* 为什么做出这个判断？
* Prompt 在哪个环节失效了？
* 哪些动作已经足够稳定，可以固化成 Skill？
没有这些记录，系统就只能不断"重来一次"；有了这些记录，系统才可能真正"学会下一次做得更好"。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

** 留痕不是为了 debug，而是为了进化。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  智能并发策略
任务拆解完成后，系统按项目分组执行：
策略  |  具体做法  |  理由
---|---|---
组间并发  |  前端任务和后端任务同时跑  |  代码在不同目录，不会冲突^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

组内串行  |  同一个前端项目的任务排队执行  |  可能修改同一文件，避免冲突^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

失败隔离  |  单个任务失败不影响其他组  |  故障不扩散^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

并行的本质不是"同时做很多事"，而是"让每件事都不需要等别人"。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  工具失败自动切换
AI CLI 工具经常遇到配额限制。我的方案是配合 Tool Prober 定时探测工具可用性：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    正常流程：task → codex（可用）→ 执行成功 → 完成^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    失败切换：task → codex（配额耗尽）→ gemini-cli → 执行成功 → 完成^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    全部不可用：task → 等待 → 5 分钟后自动探活 → 恢复后继续^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

单个工具挂了不影响整体，配额耗尽自动切换。这套机制让系统真正做到了 24 小时无人值守——从 4 个终端的手忙脚乱，到 20-30 个并发任务的稳定执行。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** 起步阶段，文件系统 + JSON 状态比数据库更适合 Agent 系统。原因很实际——出了 bug 可以直接让 AI 读文件排查，不需要教它查数据库。等系统稳定到需要事务和并发锁的时候，再升级不迟。
_ 你的 Agent 系统出了 bug，排查过程是什么样的？靠翻日志还是靠猜？  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

_
_

###  第五章：Agent 自己修了自己的 bug
花了一整天做"设计与实现对齐"之后不久，一个有意思的事情发生了。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

那天我在用 24h 打工人的需求澄清页面，发现了一个 bug：无法选择待确认问题的选项，也没有提交按钮。页面渲染出来了，但交互完全不能用。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
以前遇到这种事，流程是：打开代码 → 定位问题 → 手动修复 → 测试 → 提交。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

但这次我换了个做法——直接通过 24h 打工人自己的反馈系统提交了这个 bug。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    [用户反馈] 需求澄清页面有 bug：无法选择待确认问题的选项，也没有提交按钮。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    ↓ 系统自动触发 SDD 流程
    [spec.md] 目标：修复需求澄清页面的交互组件渲染问题^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

             验收标准：选项可点选，提交按钮可见且可用^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    ↓ AI 生成方案并拆解任务
    [plan.md] 定位：RadioGroup 组件未正确绑定 onChange 事件^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

             方案：修复组件 props 传递，补充提交按钮渲染逻辑^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    ↓ 我确认澄清结果
    [执行] 分析代码 → 定位问题 → 修改组件 → 验证修复 → 完成^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    ↓ 企微通知：任务已完成
** 它自己修复了自己的 bug。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

整个过程我只做了两件事：提交反馈、确认澄清。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

这件事背后有一个严肃的前提：  ** 自举不是凭空发生的。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

还记得 Day 15 花了整整一天做的"设计与实现对齐"吗？那一天的工作产出不只是修复了代码，更重要的是建立了三样东西： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
1. ** 清晰的设计文档  ** —— AI 知道每个模块该做什么、不该做什么。
2. ** SDD 流程  ** —— spec → plan → tasks 的标准路径，AI 按照同样的方式处理所有需求，包括关于自身的需求。
3. ** constitution.md  ** —— 架构约束文件，定义了代码组织规范、命名规则、模块边界。AI 在生成方案时会自动遵循这些约束。
没有这些基础设施，AI"自己修自己"就只是碰运气。有了这些，它才能在框架内工作，而不是自由发挥。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

从 Vibe Coding 的混乱，到一天的阵痛，到 Agent 自举——这条路的因果链非常清楚。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

** 捷径的尽头是弯路，大道的尽头是自由。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

> ** 工程建议：  ** 自举的前提是 constitution.md（架构约束文件）。不需要写得多长，但至少要覆盖三件事：目录结构约定、模块边界、命名规则。有了它，AI 才能在"框架内工作"而不是"自由发挥"。
_ 你见过最惊艳的 AI 自举案例是什么？  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


###  第六章：从 demo 到系统——门槛不是模型，是治理
做完 24h 打工人之后，我慢慢意识到一件事：  ** 留痕只是起点，不是终点。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

很多 Agent demo 的问题不是它不会跑，而是它一旦跑偏，你完全不知道发生了什么。我把这叫做"demo 跑偏时的 4 个不知道"： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
1. 是目标有歧义，还是分解有问题？
2. 是工具挂了，还是权限不够？
3. 是 Prompt 不稳定，还是系统边界不清？
4. 是这次偶然成功，还是可以稳定复现？
这四个问题回答不了，demo 就永远是 demo。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  Observability：6 个必看维度
我以前说"留痕很重要"。现在更愿意说：留痕是 debug 的起点，observability 才是系统优化的闭环。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
一个生产级 Agent 系统，至少要看得见：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    observability_dimensions:^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

      1_goal:      "当前目标是什么"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

      2_step:      "正在执行哪一步"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

      3_tool:      "用了什么工具"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

      4_failure:   "为什么失败"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

      5_recovery:  "是否触发了重试 / 回退 / 切换"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

      6_cost:      "本轮 token / 时间 / 成本消耗了多少"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

如果没有这些视图，系统一复杂，就只能靠猜。猜着猜着就不想维护了。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  Eval：4 个持续校准问题
传统软件习惯把评估理解成"上线前测一下"。但 Agent 面对的是开放环境、动态输入、不断变化的上下文。评估必须是持续的： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
评估问题  |  为什么重要
---|---
需求澄清是否稳定？  |  不稳定意味着下游全部白跑^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

任务拆解是否越来越合理？  |  拆解质量决定执行效率^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

某个 Skill 是否真的提高了成功率？  |  避免"加了等于没加"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

失败切换有没有制造新错误？  |  避免"修了一个坑，挖了两个坑"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

对 Agent 来说，  ** 评估不是验收动作，而是日常运行信号。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  Control Plane：权限、边界、审计
当系统接了更多工具、更多角色、更多目标，问题就不再是"能不能跑"，而是：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


* 谁可以调用哪些工具？
* 哪些动作默认允许，哪些必须人工确认？
* 哪些路径只能读，哪些能写？
* 哪些任务失败后必须停下来？
从 demo 到系统，中间隔着的不是更多 Prompt，而是 control plane。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  脚手架 > 模型
这是我所有原则里最反常识的一条。
24h 打工人用的不是最贵的模型。codex 配额用完切 gemini，gemini 挂了切 claude——都不是顶配。但有 SDD 流程 + 调度层 + 失败切换 + constitution.md，效果远好于直接用一次性的顶级模型。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
    投入回报对比（个人经验估算）：
    模型升级：成本 +300%，效果 +20%^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    脚手架升级：成本 +50%，效果 +200%^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    → 优先投资脚手架，而不是追最新最贵的模型
一个设计精良的系统让弱模型发挥惊人性能。反过来，烂系统完全浪费掉顶级模型的能力。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  Agent 系统的 5 层地基
如果今天让我重新概括 Agent 系统的地基，至少有五层：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
1. ** 目标表达  ** ：到底想完成什么^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
2. ** 能力单元  ** ：有哪些 Skill 工具 工作流^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
3. ** 运行时状态  ** ：当前正在做什么^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
4. ** 治理边界  ** ：允许做什么，不允许做什么^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
5. ** 评估反馈  ** ：哪些行为值得固化，哪些必须修正^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
少任何一层，系统都可能看起来能跑，但跑不稳。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 垃圾的思考乘以强大的模型，等于精美的垃圾。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** 如果你的系统还没有 observability（至少能回答"它为什么失败"），那比换一个更强的模型优先级高 10 倍。先投治理，再投模型。
_ 你觉得"脚手架 > 模型"对吗？有没有反例？  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


###  第七章：协议层正在成形
做到这里，我的视角发生了第二次升级。
以前关心的是：怎么把一个 Agent 系统搭起来。现在更关心的是：  ** 整个行业在形成哪些共识性的基础设施？  ** ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
如果只盯着单个 Agent，很容易把问题看成"Prompt + 工具 + 工作流"。但从 2025 年开始，一个更大的变化出现了：Agent 正在从单系统里的自动化，走向跨系统互操作。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
时间  |  事件  |  意义
---|---|---
2025-03-11  |  OpenAI 发布 Agents 新基座  |  Responses API + Tools + SDK + Tracing，runtime 开始收敛 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
2025-04-09  |  Google 发布 A2A  |  Agent 间协作有了协议化趋势^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

2025-06-23  |  A2A 捐给 Linux Foundation  |  从企业项目变行业标准 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
2026-02-13  |  GitHub 发布 Agentic Workflows 技术预览  |  Agent 进入 CI PR Issue 主流程 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
2026-03-16  |  Microsoft Foundry Agent Service GA  |  Responses API runtime + 全链路 tracing + 企业级 eval ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
2026-04-07  |  GitHub Dependabot → AI agent remediation  |  安全告警自动修复成为现实 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  Responses API：runtime 在收敛
OpenAI 把 Responses API、内置 Tools、Agents SDK、Tracing 组合成了更明确的开发路径。Assistants API 后续将逐步让位。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
这件事的重要性在于：Agent 开发正在从"自己拼一堆能力"，走向"围绕统一 runtime 和工具语义来构建"。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  MCP：工具接入标准化
我自己过去解决的是"怎么让 Agent 调 CLI、读文件、改代码"。MCP 解决的是更通用的问题：Agent 怎么以标准方式接入工具、资源和外部系统。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
    {
      "tool": "code_search",^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

     "description": "Search codebase by semantic query", ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
     "input_schema": {
         "type": "object",
         "properties": {
           "query": { "type": "string" },^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

           "scope": { "enum": ["current_repo", "org_repos"] }, ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
           "max_results": { "type": "integer", "default": 10 } ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
         }
      },
    "permissions": ["read:code"],^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    "rate_limit": "100/min"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    }
一旦工具接入开始标准化，重点就从"每家自己造一套胶水层"转向"怎样设计更稳定的资源暴露、权限边界和调用语义"。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  A2A：多 Agent 怎么协作
Agent 的下一个问题，不是单点更聪明，而是多 Agent 怎么协作、发现、协商和同步状态。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

这和我自己踩过的坑是同一类问题：主 Agent 上下文越来越重；子 Agent 返回后如何续跑；多角色如何共享状态而不是靠人手工搬运上下文。以前只能各自造轮子，现在开始出现协议化趋势了。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  我的判断
> Agent 开发正在从"框架之争"，转向"协议 + runtime + control plane 之争"。
这不代表框架不重要，而是它们正在往更底层、更标准化的方向收敛。对个人开发者来说，现在值得花时间理解的不是某个框架的 API，而是这些协议背后的设计理念。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 未来做 Agent，越来越像搭操作系统，不只是写 prompt。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** 选技术栈时，优先看它是否兼容 MCP / Responses API 这些正在收敛的协议。自己造的胶水层越多，未来迁移成本越高。协议层是长期资产，框架是短期工具。
_ 你在用 MCP 了吗？体验怎么样？  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


###  第八章：从 Task-Driven 到 Goal-Driven
前面讲的所有实践，回头看，本质上都是  ** Task-Driven  ** ：我先把值得做的事想好，送进系统，系统负责拆解、执行、留痕。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
这套模式非常有效。但它有一个我撞了很久才意识到的边界：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

** 24h 在线，不等于 24h 迭代。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

系统虽然能 24 小时执行，但这些更高层的判断仍然依赖我：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


* 现在最值得做什么？
* 哪个方向应该继续推进？
* 遇到阻塞时该换路径还是等待？
* 哪些问题值得主动探索？
系统在干活，但活还是我在派。  ** 只要任务还需要我持续供给，我就仍然是系统的瓶颈。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

这才是问题的根。

####  Task-Driven vs Goal-Driven
维度  |  Task-Driven  |  Goal-Driven^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

---|---|---
人的角色  |  项目经理 + 执行监督  |  目标设定者 / 审核者^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

Agent 的角色  |  执行器  |  自主推进者^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

决策中心  |  在人脑子里  |  在目标 + 边界 + 系统状态里^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

主要成本  |  人持续编排  |  前期建模和约束设计^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

适用场景  |  简单、一次性任务  |  长期、复杂、持续推进任务^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

简单说：  ** Task-Driven 解决执行问题，Goal-Driven 解决迭代问题。  ** 前者让系统开始能跑，后者才让系统开始能持续向前。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  Goal-Driven 的 5 个前提
很多人不敢放手让 Agent 自主推进，担心它跑偏、浪费 token、做无效工作。这些担心都成立。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

所以 Goal-Driven 对系统提出了更高要求：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

1. ** 目标必须清晰  ** —— 不是模糊愿望，而是可推进、可判断的目标表达。
2. ** 边界必须清晰  ** —— 哪些能做，哪些不能做，资源上限是什么。
3. ** 状态必须可见  ** —— 当前做到哪一步，卡在哪，为什么卡。
4. ** 过程必须留痕  ** —— 否则你无法知道它为什么成功，也无法知道它为什么失败。
5. ** 权限必须可控  ** —— 它到底能调用哪些工具，能写到哪里，谁来兜底。
只有在这 5 个前提成立时，自主推进才是资产；否则只会把错误放大得更快。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

Goal-Driven 不是更放权，而是更强约束下的有限自治。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


####  共享状态：STATE.yaml
Goal-Driven 一旦进入多步骤、多角色协作，主 Agent 本身会变成新的瓶颈——上下文越来越重、通信越来越慢、单点故障。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
更务实的做法是用共享状态来协调：

    # STATE.yaml — 共享任务面板
    goal: "优化搜索模块响应速度，P95 从 800ms 降到 200ms"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    owner: "joefu"
    deadline: "2026-05-01"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    constraints:

      - "不修改已有 API 契约"
      - "每日 API 成本不超过 $5"
      - "所有架构变更需记录在 changelog.md"
    agents:
      profiler:
         status: "completed"
         output: "analysis/search-perf-baseline.json"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         summary: "瓶颈定位在 DB 全表扫描和缺少缓存层"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    backend_dev:
         status: "in_progress"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         current_step: "为热点查询添加 Redis 缓存"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         blocked: false
    test_runner:
         status: "pending"
         depends_on: ["backend_dev"]^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

         description: "回归测试 + 性能压测验证"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

    next_review: "2026-04-15"^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

每个 Agent 自己读取状态、写回进度，主会话只负责高层目标和验收。主会话负责方向，不负责搬运；系统负责推进，不靠人反复传话。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

####  6 步落地路径
如果你也想开始做 Agent，我建议这个顺序：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
步骤  |  做什么  |  核心产出^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
---|---|---^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
第一步  |  写清楚 spec  |  要做什么、不做什么、怎么算完成^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
第二步  |  执行过程留痕  |  Prompt 状态 输出 / 错误全记录^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
第三步  |  补 observability 和 eval  |  知道为什么成功、为什么失败^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
第四步  |  高频动作沉淀为 Skill  |  模板 + 规则 + 代码^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
第五步  |  引入调度和并发  |  调度层 + 轮询 + 失败切换^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
第六步  |  最后才尝试 Goal-Driven  |  目标表达 + 治理边界 + 共享状态^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 先让一次执行可复盘，再让它可重复，再让它可规模化，最后让它可有限自主。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
> ** 工程建议：  ** Goal-Driven 必须建立在成熟的 Task-Driven 基础上。一个连任务都执行不稳定、没有留痕、没有 Skill 沉淀的系统，不可能真的进入目标驱动。别跳步。
_ 你的 Agent 系统现在在哪一步？卡在哪里？  _^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


###  结语：增强自我，而非取代自我
回到开头的问题：人是瓶颈。
但一年走下来，我对这句话的理解变了。
瓶颈不是人的能力不够，而是人的注意力有限。4-6 个终端是上限，不是因为不够努力，而是人脑的并发模型就长这样。解决方案不是"让 AI 替代人"，而是"让系统不再依赖人的实时在场"。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
24h 打工人的 SDD 是这个系统的地基：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


* ` spec.md  ` 把模糊的需求变成明确的目标
* ` plan.md  ` 把目标变成技术方案
* ` tasks.md  ` 把方案变成可执行的步骤
* ` constitution.md  ` 把经验变成可复用的约束
* ` eval / trace / policy  ` 把系统变成可观测、可治理、可持续迭代的能力体
Goal-Driven 是它的下一站：让系统不只是等你派活，而是围绕目标自主向前走。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

回顾整条路——从 4 个终端的手忙脚乱，到 Vibe Coding 翻车后的痛定思痛，到 24 小时无人值守的稳定执行，到 Agent 自己修自己的 bug，再到协议层正在收敛的行业共识——每一步的认知转折都不是提前设计好的，是被实践逼出来的。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
** 真正的跃迁，不是让 AI 多做几个步骤，而是让人退出微观调度。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

** 增强自我，而非取代自我。共勉。  **^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


## 深度分析
### 1. 核心命题的演进路径
这篇文章本质上是一个工程师的**认知迭代复盘**，展现了从直觉到系统思维的完整演进过程。作者的思考可以归纳为四个阶段： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
**第一阶段（人是瓶颈）**：认识到多 AI 终端并发的物理极限——人脑 context switch 上限为 4-6 个并发窗口。这个认知打破了对"AI 并发=效率"的简单假设。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
**第二阶段（工具选择的经济学）**：提出"代码优先于 Prompt"的决策框架，本质上是成本-收益分析。不确定性越高、成本越高的方案，越需要明确的场景限定。这与软件工程中"简单方案优先"的原则一脉相承。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
**第三阶段（系统化工程）**：从 Vibe Coding 的"先易后难"翻车中顿悟 SDD（Spec-Driven Development）的价值。这一步的认知转折最为关键——作者意识到 AI 生成的代码在没有规格约束的情况下会快速熵增，最终需要"设计与实现对齐"的补课。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
**第四阶段（治理与演进）**：从单点工具走向平台思维，提出 observability、eval、control plane 等生产级要素，以及 Task-Driven 向 Goal-Driven 的范式跃迁。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

### 2. 技术路线的选择逻辑
作者选择 **CLI 而非 API** 作为调度层的基础，这个决策并非教条，而是阶段性的最优解：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


- CLI 工具有内置的上下文管理、工具调用和错误处理，降低了自研成本
- 文件系统 + JSON 状态管理在快速迭代期更易于 debug 和版本控制
- 等系统稳定到需要事务和并发锁时，再升级数据库
这种"够用就好，逐步演进"的思路，避免了过度设计，同时为未来留足了升级空间。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


### 3. 自举的技术前提
Agent 修复自己的 bug 这一场景看似惊艳，但作者清醒地指出：自举不是凭空发生的。它需要三块基础设施的支撑： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

- **设计文档**：让 AI 知道每个模块的边界
- **SDD 流程**：让 AI 按统一方式处理包括自身在内的所有需求
- **constitution.md**：架构约束文件，防止 AI"自由发挥"
这揭示了一个重要原则：**AI 的自主能力与系统的基础设施完备程度正相关**。没有这些约束，自主只会放大错误。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

### 4. 行业协议层的信号意义
作者观察到 2025-2026 年间 MCP、A2A、Responses API 等协议层的收敛，并判断 Agent 开发正在从"框架之争"转向"协议 + runtime + control plane 之争"。这一判断的战略含义是： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

- 框架是短期工具，协议是长期资产
- 未来的 Agent 系统竞争力在于**基础设施的完善度**，而非使用哪个框架
- 类似操作系统的发展规律：先有硬件和协议栈，才有应用生态

### 5. Task-Driven 与 Goal-Driven 的本质区别
| 维度 | Task-Driven | Goal-Driven |
|------|-------------|-------------|
| 人的角色 | 项目经理 + 监督者 | 目标设定者 + 审核者 |
| 系统瓶颈 | 执行效率 | 目标推进的自主性 |
| 核心挑战 | 让任务稳定完成 | 让目标持续演进 |
| 前提要求 | 5 层地基必须完备 | Task-Driven 成熟后才可能 |
Goal-Driven 本质上是一种**有限自治**：在清晰的目标、边界、状态、留痕和权限约束下，Agent 可以自主推进而不需要人实时介入。这代表了 AI Agent 从"工具"向"协作者"的转变。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

## Related entities
- [[entities/ai-agent-exploration-path-legacy-tech|十年老技术开发的 AI Agent 探索之路]]
- [[entities/ai-agent-exploration-path-legacy-tech.md|十年老技术开发的 AI Agent 探索之路]]
- 十年老技术开发的 AI Agent 探索之路
- [[entities/十年老技术开发的-ai-agent-探索之路-v2|十年老技术开发的 AI Agent 探索之路]]

## 实践启示
### 起步原则：先记录，再造系统
如果现在正在手动管理多个 AI 终端，**不要急于造系统**。先记录一周：哪些操作是重复的？哪些切换是可以消除的？瓶颈清单比技术方案更重要。这是避免过度工程的第一步。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

### 决策层级：自下而上评估
面对每个新需求，按以下顺序评估：
1. **10 行 Bash 能搞定吗？** — 能用脚本解决的不要上 AI
2. **一次 LLM 调用够吗？** — 简单语义任务不需要 Agent 循环
3. **最后才考虑 Agent** — 多步推理、动态决策、循环执行的场景才用
这个习惯可以帮你省掉 80% 的过度工程。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


### Vibe Coding 的止损点
如果正在 Vibe Coding，享受前三天的快感没问题，**第三天必须开始补 spec**。越早补，代价越小。哪怕只有三段话——要做什么、不做什么、怎么算完成。 ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]

### 24h 系统的最小架子
文件 + 轮询 + JSON 状态管理是 Agent 调度层的最小可行架构。核心模块：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


- **文件队列**：接收任务输入
- **轮询调度**：分发执行
- **JSON 状态**：记录输入输出
- **Tool Prober**：探测工具可用性，自动切换
起步阶段不要上数据库——出了 bug 可以直接让 AI 读文件排查。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


### 自举的三个前提
如果希望 Agent 能自主修复自身问题，必须提前建立：^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


- **设计文档**：每个模块的职责边界清晰
- **SDD 流程**：spec → plan → tasks 的标准路径
- **constitution.md**：目录结构、命名规则、模块边界的架构约束

### 脚手架优先于模型
投入回报的经验估算：

- 模型升级：成本 +300%，效果 +20%
- 脚手架升级：成本 +50%，效果 +200%
**优先投资脚手架，而不是追最新最贵的模型。** 一个设计精良的系统让弱模型发挥惊人性能。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


### 治理优先于模型
如果系统还不能回答"它为什么失败"，那比换一个更强的模型**优先级高 10 倍**。先投治理，再投模型。治理的四层： ^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
1. **Observability**：至少能回答"它正在做什么、为什么失败"
2. **Eval**：不是上线前测一下，而是日常运行信号
3. **Control Plane**：权限、边界、审计
4. **Skill 沉淀**：高频动作固化为模板和规则

### 6 步落地路径
不要跳步。按以下顺序演进：
1. 写清楚 spec
2. 执行过程留痕
3. 补 observability 和 eval
4. 高频动作沉淀为 Skill
5. 引入调度和并发
6. 最后才尝试 Goal-Driven
**先让一次执行可复盘，再让它可重复，再让它可规模化，最后让它可有限自主。**^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]


### 技术栈选择
选技术栈时，优先看它是否兼容 **MCP / Responses API** 这些正在收敛的协议。自己造的胶水层越多，未来迁移成本越高。协议层是长期资产，框架是短期工具。^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
---^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]
→ [[raw/articles/十年老技术开发的-ai-agent-探索之路|原文存档]]^[raw/articles/十年老技术开发的-ai-agent-探索之路.md]