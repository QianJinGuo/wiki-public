---

title: "小白也能搞懂：.claude/ 文件夹里到底应该放什么"
type: entity
created: 2026-07-04
updated: 2026-09-26
tags: [wechat, ai]
rating: v8c7
sources:
  - raw/articles/小白也能搞懂claude-文件夹里到底应该放什么
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 小白也能搞懂：.claude/ 文件夹里到底应该放什么

**来源**: 架构师

**发布日期**: 2026-04-23^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


**原文链接**: https://mp.weixin.qq.com/s/v6iBuqep2ZCZ9wAwhnHLDA ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

---


架构师（JiaGouX）

我们都是架构师！

架构未来，你来不来？

今天来聊一个大家可能天天见，但不一定真正用好的东西：^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


CLAUDE.md  和  .claude/  。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


如果你已经用过 Claude Code，大概率见过它们。可能是  /init  自动生成过一个  CLAUDE.md  ，也可能是在项目根目录下看到过  .claude/settings.json  、  .claude/commands/  、  .claude/agents/  这些东西。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

但很多人对它们的用法，仍然停在一个很模糊的印象里：^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


"这不就是 Claude 的配置吗？"

我以前也差不多这么看。最近连着梳理  Agent 最小闭环  、  Harness  、  上下文管理  、  Prompt Caching  ，回头再看这堆文件，才慢慢觉得它们没那么简单。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

这些文件不是给 Claude "加魔法"的。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


它们更像是在回答一个很朴素的工程问题：

项目里有没有一套东西，能让 Agent 少靠猜。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


少猜这个项目怎么跑。少猜哪些目录不能碰。少猜团队代码风格是什么。少猜出了错该怎么验证。少猜什么动作是危险动作。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

把前面几篇连起来看会更清楚：  Agent 最小闭环讲的是模型怎么动起来  ；  Harness 讲的是模型外面的运行系统  ；  上下文管理讲的是什么该留  、什么该丢；  Prompt Caching 讲的是稳定前缀和动态尾部怎么分层  。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

那今天这篇，就把这些东西落回一个最具体的问题：^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


一个普通项目里，  .claude/  文件夹到底应该放什么？^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


先说我自己的理解：

它不该是第二个文档站，也不该是提示词垃圾桶。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


它更像一张给 Agent 的项目工作台——左边放稳定规则，中间放权限和工具边界，右边放可复用流程。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]


## 太长不看版

- •
  CLAUDE.md
  先放最稳定、最高频、最影响行为的项目规则：命令、架构边界、目录职责、测试方式、危险区。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- •
  .claude/CLAUDE.md
  和
  .claude/rules/.md
  在 2.1.88 源码里仍然是项目记忆加载路径；官方文档更强调^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

  CLAUDE.md
  、导入和
  /memory
  视图。写生产配置时，以当前版本实际加载结果为准。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- •
  .claude/settings.json^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

  放权限边界：哪些命令能跑，哪些文件不能读，哪些动作必须先拦住。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- •
  .claude/hooks/
  放确定性动作：危险命令拦截、写后格式化、结束前测试。提示词只能提醒，hooks 能真正执行。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- •
  .claude/commands/
  或 skills 放重复工作流：代码审查、修 issue、生成发布说明、安全检查。不要把它当提示词收藏夹。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- •
  .claude/agents/
  放需要独立上下文的专家角色：代码审查、安全审计、性能排查。重点是隔离中间过程，不是凑"多 Agent 团队"。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- • 个人偏好放
  ~/.claude/
  或本地配置，团队契约放项目仓库。别把自己的习惯提交成全队规则。 ^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

- • 还有一条我自己踩过坑才想明

^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

→ [[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么|原文存档]]

---

## 深度分析

### 五层分层：规则、权限、动作、流程、隔离的职责切分

文章最有价值的框架不是目录清单，而是分层逻辑：CLAUDE.md 管的是"应该怎么做"的软约束，settings.json 管的是"允许做什么"的硬边界，hooks 管"做了以后自动发生什么"的确定性动作，commands/skills 沉淀可复用流程，agents 则把支线探索隔离到独立上下文。这些层不能混——最典型的反面教材是"不要读取 .env"只写在 CLAUDE.md 里：模型遵不遵守全靠自觉，作者亲历 Claude 在 debug 时顺手读了 .env，改成 settings.json 的 deny 规则才真正堵住。这背后是 Harness 视角的判断：Agent 生产级的差距，不在模型会不会说，而在动作能不能被看见、被约束、被验证。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

分层还延伸到仓库边界：项目级 `.claude/` 与用户级 `~/.claude/` 是两套体系，判断标准只有一条——团队契约进仓库，个人习惯留本地。"默认用中文回复"提交成团队配置会改变队友那边 Claude 的行为；"数据库迁移必须人工确认"才是团队契约。作者还踩过一个实际坑：在 `~/.claude/CLAUDE.md` 写了语言偏好后，英文项目里 Claude 也开始说中文——项目级 CLAUDE.md 若未显式覆盖，用户级配置就会漏进合并结果。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

### exit 2 才是真正的安全门：hooks 的确定性语义

hooks 部分最关键的技术事实是 exit code 的语义差异：exit 0 成功放行，exit 1 只是非阻塞错误（工作流继续，根本没拦住），只有 exit 2 才真正阻断执行，且 stderr 会发回给 Claude 自我修正。作者翻了 2.1.88 源码验证，"用 exit 1 拦危险命令"正是 Akshay 总结的最常见错误。Stop 门禁还藏着死循环陷阱：必须检查 JSON payload 里的 `stop_hook_active` 标志，第二次尝试必须放行。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

hooks 的心智模型是职责拆分：模型提出动作，系统独立裁决，危险动作被确定性拦截。但 hooks 以你的系统权限运行、没有沙箱——不检查 JSON 输入就拼进 shell 命令、路径不用绝对路径，都会把 hook 变成攻击面。性能上也有隐性代价：hooks 在 subagent 动作里同样触发，作者曾因 PostToolUse 重 hook 拖慢所有子任务。所以只配三类：PreToolUse 快速安全判断、PostToolUse 轻量格式化、Stop 最小质量门禁，复杂扫描交给 CI。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

### CLAUDE.md 与稳定前缀的隐性耦合：写得越多，遵从度越低

CLAUDE.md 的内容会进入系统提示的稳定前缀区域——正是 KV Cache 最想命中的那一段。这决定了两条隐性纪律：动态内容（日期、任务进度、随机 ID）绝不能进 CLAUDE.md，否则等于自己弄脏最该稳定的前缀、破坏缓存命中；长文档、完整接口文档、formatter 已能表达的格式细节也不该塞进来，既浪费上下文窗口又稀释关键规则的权重。作者赞同 Akshay 的"200 行以内"建议：文件越长，写在后面的规则越容易被"遗忘"。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

另一个被多数教程讲错的边界是 `.claude/rules/`：2.1.88 源码的 `claudemd.ts` 确实读取该目录，但官方文档更强调 CLAUDE.md、`@path` 导入和 `/memory` 视图。作者曾写了 rules 文件结果 Claude 根本没加载——"我在某篇教程里看过"不够用，进仓库的规则必须能被团队确认、版本管理、回滚，并用 `/memory` 或调试日志确认真实加载。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

### commands 向 skills 收敛，agents 的价值在上下文隔离而非"多角色"

commands 常被当"提示词收藏夹"，但正确定位是团队流程入口：`!` 反引号语法在 prompt 发出前先执行 shell 并嵌入输出，让每次代码审查都从真实 diff 开始；`$ARGUMENTS` 让 /project:fix-issue 234 自动注入 issue 内容。官方口径里 custom commands 已向 skills 收敛——skills 可带目录、模板、脚本，且能根据对话上下文自动触发（command 必须手动输入）。取舍：轻的固定入口用 command，带模板和参考资料的复杂流程用 skill。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

agents 层最容易被讲歪：subagent 的核心价值不是凑"多 Agent 团队"，而是独立上下文、独立工具权限。代码审查、安全审计产生大量中间噪音（grep 结果、失败假设、长日志），留在主上下文会弄脏后续工作记忆——subagent 让探索留在子上下文，主线程只收压缩结论。agent 定义要写清三件事：何时用、能用哪些工具、返回什么格式。权限限制是刻意的：security-auditor 只给 Read/Grep/Glob 不给写权限，只报告不修复；只读型 subagent 可用更便宜的模型跑。^[raw/articles/小白也能搞懂claude-文件夹里到底应该放什么.md]

## 实践启示

1. **从五步走开始，别一次配满**：先写 ≤100 行的 CLAUDE.md（/init 生成后砍掉一大半），再补只允许读/搜索/diff/测试的保守 settings.json，然后加 Bash 防火墙和 Stop 门禁两个 hooks，高频流程才做成 command/skill，最后等支线变重再上 agents。95% 的项目前三步就够。

2. **能机器验证的规则，别指望 CLAUDE.md 的自觉**：.env 用 settings.json 的 deny 拦、代码风格用 PostToolUse hook 跑 formatter、"完成了"前必须过测试用 Stop hook 的 exit 2 强制。CLAUDE.md 只放"每次都值得带上"的稳定规则。

3. **写安全 hook 前先背下 exit code 语义**：拦截必须是 exit 2（stderr 回传给模型自我修正），exit 1 只是看起来报错、实际放行；Stop 门禁必须检查 `stop_hook_active` 防止拦截-重试死循环。

4. **给 CLAUDE.md 做减法而不是加法**：动态状态、时间戳、完整文档、个人偏好一律不放；规则多时拆外部文档用 `@path` 导入；配置完用 `/memory` 视图确认加载结果，尤其 `.claude/rules/` 这类文档与源码口径不一致的路径。

5. **用"团队契约还是个人习惯"一刀切仓库边界**：CLAUDE.local.md 和 settings.local.json 保持 gitignored；语言偏好、本机脚本路径留 `~/.claude/`；别让一个人的配置改变全队 Claude 的行为。

6. **subagent 只用于窄权限 + 大量中间态的任务**：定义写清触发时机、工具白名单（审计类只读不给写）和输出格式；需要模板、脚本、参考资料的工作流用 skill 而不是堆长 prompt。

---

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

