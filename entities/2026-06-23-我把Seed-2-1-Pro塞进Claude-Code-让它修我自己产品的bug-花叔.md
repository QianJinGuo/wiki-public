---
title: "我把Seed 2 1 Pro塞进Claude Code 让它修我自己产品的bug 花叔"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-23-我把Seed-2-1-Pro塞进Claude-Code-让它修我自己产品的bug-花叔]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-23-我把Seed-2-1-Pro塞进Claude-Code-让它修我自己产品的bug-花叔.md|原文存档]]

sha256: 16c59e73ac4db124d173790ee2b373a1f50e89bc1793b1adc8829ebdf2fdfc49 ^[raw/articles/2026-06-23-我把Seed-2-1-Pro塞进Claude-Code-让它修我自己产品的bug-花叔.md]

## 摘要

博主花叔在豆包大模型 2.1（即 Seed 2.1 Pro）发布当天，把它接入自己天天使用的 Claude Code 工作流做实测：配置只需三步（火山方舟开通 doubao-seed-2-1-pro-preview、拿 ARK API Key、设 ANTHROPIC_BASE_URL/AUTH_TOKEN/MODEL 三个环境变量指向兼容 Anthropic 协议的端点），并在 .zshrc 里写 doubao 别名与原 claude 命令互不干扰。测试一是修自己开源产品 FanBox（Coding Agent 驾驶舱，自写代码 28 个文件 15609 行，主逻辑 app.js 达 4572 行，一周多 97 次提交从 v1.1 滚到 v2.3）的两个真实 GitHub issue：#27 终端复制粘贴失效、#28 新增 skills 加载不出来。Seed 2.1 Pro 的表现是"先探索、再规划、最后才动手"——自动进 plan mode、并行派两个 Explore 子 agent（一个 45 次工具调用 12 万 token 啃终端实现、一个 35 次近 4 万 token 查 skills 机制）、再起 Plan agent 花十来分钟设计方案，然后在 auto mode 里一口气自己跑了 40 多分钟完成两个 issue，中间无需人工干预。^[raw/articles/2026-06-23-我把Seed-2-1-Pro塞进Claude-Code-让它修我自己产品的bug-花叔.md]

修复质量上，#27 顺着项目原本写法装 xterm 官方剪贴板插件、挂按键处理（Cmd+C 复制选中、Cmd+V 读系统剪贴板用 bracketed paste 安全塞入，顺手补了 Cmd+加减调字号和右键菜单选中即复制对齐 iTerm2），改动落在 5 个文件 80 多行，并照仓库原有 __noXterm 风格加了 __noClipboard 兜底；#28 定位到出乎意料的根因——skills 加载只扫"最近 12 个活跃项目"，新项目落在名单外，它加了强制刷新接口绕过缓存。测试二是给一段 10 秒左右的 Stripe 中文官网滚动录屏（不给图），Seed 2.1 Pro 与 Opus 4.8 都能抽帧理解后复刻出"会动"的网页（滚动淡入、渐变飘动），Seed 2.1 Pro 认得出顶部导航、Hero 文案、橙粉紫渐变和 bento 卡片区，但复杂区块的流体渐变只抓到颜色没抓到形态。作者结论：Seed 2.1 Pro"确实上牌桌了"，属国内第一梯队，与海外顶尖（Opus 4.8）仍有距离；对照 Arena Code Arena: Frontend 盲投榜，它排第 8、1539 分与 Opus 4.6 同档，与体感吻合。^[raw/articles/2026-06-23-我把Seed-2-1-Pro塞进Claude-Code-让它修我自己产品的bug-花叔.md]

## 关键要点

- 测试时机有背景：Anthropic 最新旗舰 Fable 5 发布三天即被美国政府以出口管制为由叫停，国产模型在海外越来越出圈，榜单上多家国产模型在列。
- FanBox 项目由 Fable 5 起头（6 月初开工），架构含 Electron 无构建运行时、node-pty 真终端、xterm.js 渲染、Monaco 编辑器，还有微信 ClawBot、上下文自动整理、记忆层兜底。
- 作者强调"能长时间稳住复杂任务本身就是能力"：差一点的模型跑十几分钟就丢上下文或草草交差，能连跑一小时不散架的往往才好用。
- 字节在发布材料中把"优化 Harness 与模型的协同"列为下一步方向，说明模型厂商也在认真对待"模型配不配合 harness"这件事。
- 视频复刻测试的价值论证：喂静态图最多还原样子，喂录屏能把滚动、hover 反应、流动渐变等动效一起复现——视频理解正是 Seed 2.1 这次重点讲的能力。

## 来源

- 原文: [[raw/articles/2026-06-23-我把Seed-2-1-Pro塞进Claude-Code-让它修我自己产品的bug-花叔.md|我把Seed 2 1 Pro塞进Claude Code 让它修我自己产品的bug 花叔]]
