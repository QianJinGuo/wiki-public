---
title: "Claude Mods：把 Agent Harness 翻转成 Attention Interface"
created: 2026-10-22
updated: 2026-10-23
type: entity
tags: [agent, harness, harness-engineering, claude-code, attention-interface, plugin, middleware]
sources: [raw/articles/claude-mods-attention-interface-harness-reversal-2026]
confidence: 0.7
provenance_state: extracted
---

# Claude Mods：把 Agent Harness 翻转成 Attention Interface

Latent Space 作者 Dan McAteer 在 2026-10-21 发表《Claude Mods: Flipping the Harness Inside Out》，把 Claude Code 的 Mods 机制（TypeScript/JavaScript 函数形式的插件）提炼为 **Attention Interface（注意力接口）** 这一概念：harness 的朝向从"面向机器"翻转为"面向人"。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

## 核心论点：harness 的朝向翻转

原始 agent harness 是 **agent 与机器之间的接口**——用户发 prompt，agent 执行命令、编辑文件、调用工具。当模型把越来越多 harness 能力吸收进权重之后，**剩下的那部分正对着人**，它决定 agent 何时打断你、给你看什么、需要你做哪些决定，这就是 Attention Interface。Claude Mods 是该接口的早期形态，而且是用户可自己塑形的一版。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

作者的赌注是：**harness 最终会变成平台**，其它公司在上面构建应用，模型成为引擎——类似浏览器成为 web app 平台。"Mods make the human layer programmable, by the human who uses Claude." 这与 [[concepts/harness-as-product-surface]] 的判断同向，也是 [[concepts/harness-engineering]] 从"工程实践"走向"人机交互层"的一条分支。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

## Mods 的工程形态

一个 mod 就是 **Claude Code 插件**，由 TypeScript/JavaScript 函数组成，运行在 Claude Code 内部（CLI 与 Desktop），可响应 prompt、tool call、界面渲染等事件，观察事件、改写渲染方式、或直接接管默认行为——本质是 **agent harness 的 middleware**。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

Anthropic 已内置六个 mod（`/plugin` → Installed → Built-in）：`agents-md`（加载 AGENTS.md 作为项目指令）、`diff`（接管 /diff 并绘制面板）、`plugin-authoring`（给 Claude 写 mod 的 skill）、`sec-default`（保护组织受管内容）、`telemetry`（发送埋点）、`you-should-know`（旁路 side-agent，默认关闭）。值得注意的是多数 mod 在后台工作——说明 Anthropic 本身也在用 mod 形态构建 Claude Code 的部件。Claude Code 内置 skill 可教 Claude 自己写 mod，用户用自然语言描述即可生成。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

作者最常用的两个：**You Should Know**（旁路 side-agent 观察主 agent，标记容易被淹没的关键动作；实测案例是它发现 CompareView.swift 及其余约 45 个文件存在未提交改动，提醒先 commit/stash，避免了演示改动回滚时误删）与 **Next Steps Supervisor**（只在"编辑了文件/跑了命令/≥8 次 tool call"的轮次后运行，给出可点击的下一步建议；作者封装成开源插件 ModSmith 发布）。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

## 与另两种 Attention Interface 的对比

作者用三个系统对照"注意力接口由谁设计"：^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

| 系统 | 谁设计接口 | 形态 |
|------|-----------|------|
| ChatGPT Intelligent UI（2026-10-07 随 GPT-6 发布） | **模型**逐条答案生成界面（图表/地图/清单/表单/小工具），在实验室给定的组件库框架内 | 零配置，控制力低 |
| Cursor PR Routing & Approval | **团队**写策略（code ownership + commit 历史分派 reviewer、按风险分自动批准、结合 Bugbot/Security 结论与仓库内政策文件），固定评审流程执行 | 可配置，但交互流程是 Cursor 的 |
| Claude Mods | **用户与 agent 一起在代码里构建接口本身** | 可钩住 loop 任意点，代价是自行负责其有用性 |

其中一条设计细节值得记录：Cursor 规定若某 PR 修改了审批策略，agent **不得用被改后的版本**放松该 PR 的评审，政策一律从 base branch 读取——即 agent 不能改规则让自己的 PR 过审。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

## 已知代价（作者自述的三个坑）

1. **噪音**：Next Steps Supervisor 的触发条件（编辑文件/跑命令/≥8 tool call）几乎覆盖每一轮——作者自 10 月 15 日安装后 111 轮里有 99 轮达标，部分 verdict 是噪音，几乎每轮都插话会导致用户开始略过它，反而消解了 Attention Interface 的意义。
2. **成本**：每次 supervisor 检查都要 fork 会话并重读上下文，大部分命中 prompt cache，但 mod 自身 README 估计 50k token 会话下每次检查约 5–6k input-equivalent token，cache 失效时更高。
3. **信任**：mod 是以用户权限运行的代码，加载后可读写文件、看到所有 prompt、消耗额度，且 **mods aren't sandboxed**。^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md]

## 深度分析

### 翻转的机械含义：接收端从机器换成了人

Mods 的形状并不新：TypeScript/JavaScript 函数挂到 prompt、tool call、渲染事件上，可观察、可改写、可顶替默认实现^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:41-50]，就是 middleware。变的是输出端。同一份 middleware 输出到 shell，校验标准是"命令是否成功、文件是否落在预期状态"，可枚举可回放；输出到人，标准变成"这条是否值得占用一个人三秒钟"，带宽只有每秒几比特。

所以翻转翻的是**反馈回路的接收端**：从把 agent 意图译给机器，转为把 agent 内部状态译给人。机器接口失败会抛错，注意力接口失败不抛错，只表现为用户开始略过——退化是静默的，度量得自造。模型把可内化的能力吃进权重后，剩下的恰是不可内化的授权与时机：何时打断、给谁看什么、哪个决定必须由人签字^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:30-36]。于是 harness 的边际成本从工程实现迁到人的注意力，而后者不可 scaling，竞争从功能转向单位注意力的转化率。

### 对比轴重画：三种接口分配的是设计权

按"谁设计"排表是对的，但可拆成四维。**粒度**：Intelligent UI 逐条答案、模型在实验室组件库内选型；Cursor 逐类 PR、团队写 policy 而流程固定；Mods 任意 loop 点、用户与 agent 一起写代码^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:137-145]。**时点**：生成中／评审前／回合结束。**可撤销性**差别最大：只有 mod 能 `/supervisor off` 或改触发条件^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:168-171]。**信任边界**：Cursor 规定改审批策略的 PR 不得用改后版本放松自身评审，一律读 base branch^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:129]，即"被审对象不能改审判规则"写进系统；mod 则无沙箱^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:161-167]，风险与 [[concepts/agent-sandbox]] 同层。

四维合看，三者不是同一产品的三种实现，而是把设计权分别交给模型、组织、个人。Mods 独特性不在"更可配置"，而在接口本身成为用户的产物，代价随之转移：你负责让它有用^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:148]。推论是设计权与责任必须同址——ChatGPT 收走设计权，也收走了"这屏是否有用"的责任。

### 三个代价共用一个结构：开销全记在用户账上

噪音：111 轮里 99 轮触发，用户开始略过^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:155]。成本：每次检查 fork 会话重读上下文，50k 会话约 5–6k input-equivalent，即 [[concepts/subagent-spawning-pattern]] 的固定开销在旁路观察场景复现。信任：能读写文件、看全部 prompt、花额度且不在沙箱内，built-in mod 在 `--safe-mode` 下仍运行^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:157-167]。三者分别吃掉注意力、额度、权限，都是用户的资源。

根因是触发条件错位：用"编辑文件／跑命令／≥8 tool call"这类**活动量代理**逼近"此刻需要人判断"，而活动量与不确定性只是弱相关，于是九成命中。信号本该在 agent 侧——是否重试、是否自我矛盾、是否给出"已解决但同时列两个缺口"这种内部不一致（作者 session 确有此例^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:177]）。可泛化的是 middleware 挂载点、不确定性触发、接口必须可关；不可泛化的是"用户写代码"这一前提，因为 Intelligent UI 的目标用户本就不是工程师^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:139]。再往前，harness 自读 transcript 找摩擦^[raw/articles/claude-mods-attention-interface-harness-reversal-2026.md:180-186] 是 [[concepts/agent-self-improvement-loops]] 的温和形态，却把"什么算摩擦"的定义权交了出去。

## 实践启示

1. **用不确定性触发，别用工作量触发。** 让 agent 回合末输出固定字段自评（重试次数、是否推翻先前结论、是否有未验证假设），只在标记"有分歧"时唤起旁路检查。
2. **给每个 mod 设注意力预算与自动降级。** 每 N 轮最多插话一次，同一建议连续 3 次被跳过则静音并需重新授权——用户一旦习惯略过，后续有效信号也读不到。
3. **先定权限再写接口。** 第三方 mod 逐行读源码、pin 版本；注意 `--safe-mode` 不隔离内置 mod；敏感仓库里 mod 应与 CI secret 同级，而不是当作界面美化。
4. **用作者三问当 KPI 并定期删 mod。** 是否 surfaced 我本来会漏的东西、是否让某个决定更容易、是否让我不必读更多输出；三条都不中就是 window-dressing。
5. **自建 harness 先做最小切片。** 只在回合结束点挂一个钩子＋side-agent＋字段固定的结构化 verdict，命中率稳定后再扩挂载点。

## 关联

- [[concepts/harness-as-product-surface]] — harness 作为产品/平台面的同一判断
- [[concepts/harness-engineering]] — 本实体是其"人机交互层"分支的具体案例
- [[concepts/claude-code-deep-architecture-analysis]] — Claude Code 内部架构与扩展点
- [[entities/claude-dispatch-and-the-power-of-interfaces]] — 接口层价值的另一种论证
- [[entities/interface-commoditization-ai-era]] — 接口商品化视角的对照

→ [[raw/articles/claude-mods-attention-interface-harness-reversal-2026|原文存档]]
