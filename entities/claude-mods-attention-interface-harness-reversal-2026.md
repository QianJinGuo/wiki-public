---
title: "Claude Mods：把 Agent Harness 翻转成 Attention Interface"
created: 2026-10-22
updated: 2026-10-22
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

## 关联

- [[concepts/harness-as-product-surface]] — harness 作为产品/平台面的同一判断
- [[concepts/harness-engineering]] — 本实体是其"人机交互层"分支的具体案例
- [[concepts/claude-code-deep-architecture-analysis]] — Claude Code 内部架构与扩展点
- [[entities/claude-dispatch-and-the-power-of-interfaces]] — 接口层价值的另一种论证
- [[entities/interface-commoditization-ai-era]] — 接口商品化视角的对照

→ [[raw/articles/claude-mods-attention-interface-harness-reversal-2026|原文存档]]
