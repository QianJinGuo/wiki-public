---
title: "AutoHarness 走到哪了：学界业界的能力调研"
created: 2026-07-28
updated: 2026-09-07
type: entity
tags: ['auto-harvested']
sources: [raw/articles/auto-harness-survey-vibecoder-2026]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/auto-harness-survey-vibecoder-2026.md|原文存档]]

- TextArena 145 个游戏中合法动作率达 100% ^[raw/articles/auto-harness-survey-vibecoder-2026.md]

## 摘要

这是一篇关于自动 Harness 演化（Automated Harness Engineering）的学界业界调研，核心论点是 Agent 的下一轮竞争正从模型能力转向 Harness 能力（涵盖 memory、skills、工具描述、middleware、planner、沙箱、权限、发布与回滚）。调研覆盖 DeepMind AutoHarness（把环境规则编译成 Python 代码，TextArena 145 个游戏合法动作率 100%）、Google ReasoningBank（经验记忆，Gemini-2.5-Flash 在 WebArena +8.3pp、SWE-bench Verified +4.6pp）、AHE（搜索 system prompt/memory/skills/tools/middleware，Terminal-Bench 2 pass@1 从 69.7% 升至 77.0%）以及 HARBOR、TTHE、Adaptive Auto-Harness、Meta-Harness 等方案，并给出"两条循环（演化循环/适配循环）+ 三层权限（候选生成器、独立记分器、不可变安全内核）"的生产架构、八周落地计划与四条安全红线。文章同时引用 Anthropic Retro Game Maker（6 小时 $200）与 OpenAI 内部产品（约 5 个月、约 100 万行代码、约 1500 个 PR）作为 Harness 工程进入真实开发流程的证据。^[raw/articles/auto-harness-survey-vibecoder-2026.md]

## 关键要点

- AutoHarness (DeepMind)：环境规则编译成 Python，TextArena 145 个游戏合法动作率 100%，16 个单人游戏 Harness-as-Policy 平均奖励 0.870；能判定动作合法性但无法自动保证任务成功
- ReasoningBank (Google)：演化对象是经验而非 Harness 本身，要求记录来源、适用范围、模型版本、过期时间与反例，避免错误记忆长期污染
- AHE：Terminal-Bench 2 pass@1 69.7%→77.0%；SWE-bench Verified 75.2%→75.6% 但 tokens/trial 下降约 12%——Harness 价值有时体现在更少上下文与更稳定执行；缺口是归因记录不等于因果证明、无原子发布闭环
- 生产架构：演化循环（低频、受限写权限）与适配循环（请求路径、默认只读）分离；候选生成器、独立记分器、不可变安全内核三层权限
- 四条安全红线：不让 evolver 修改身份/权限/密钥/网络出口；不让 proposer 读取 hidden holdout；不让生成候选的模型自行裁定有效；不根据单一 benchmark 直接全量发布
- 真实案例：Anthropic Retro Game Maker 连续迭代 6 小时花费 $200（solo 仅 20 分、$9）；OpenAI 内部产品约 5 个月 100 万行代码 1500 个 PR

## 来源

- 原文: [[raw/articles/auto-harness-survey-vibecoder-2026.md|AutoHarness 走到哪了：学界业界的能力调研]]
- 原始链接: : "https://mp.weixin.qq.com/s/J9FKvVCUjfgj_N3uE35GwQ
