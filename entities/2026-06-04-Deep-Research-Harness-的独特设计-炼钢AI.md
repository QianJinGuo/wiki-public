---
title: "1.前言"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-04-Deep-Research-Harness-的独特设计-炼钢AI]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-04-Deep-Research-Harness-的独特设计-炼钢AI.md|原文存档]]

sha256: 12f24d9c42c1c6be47f52c3e47c55735dc6971140f6e6105ca55c2829903ef8c ^[raw/articles/2026-06-04-Deep-Research-Harness-的独特设计-炼钢AI.md]

## 摘要

炼钢AI 公众号深度解析 MiroFlow（MiroThinker 产品背后的 deep research agent 脚手架）的独特设计，与 Claude Code 这类 code agent 脚手架逐项对比。文章先指出两类任务的本质差异：code 目标清晰可验证、操作本地文件系统、5–50 个工具调用、错误成本高；deep research 目标模糊、操作不可控外部世界、可能需要几百轮操作、错误几乎无副作用但需要更强容错。MiroFlow 的关键设计包括：当天日期硬编码进 system prompt（避免模型按训练 cutoff 判断"今年"写出诡异 query）、强制每次 response 只返回一个工具调用（与 Claude Code 鼓励并行调用相反，因为 deep research 的工具调用之间有强依赖，必须把每步中间结果纳入推理）^[raw/articles/2026-06-04-Deep-Research-Harness-的独特设计-炼钢AI.md]

工具层：MiroFlow 不用原生 function call，而是在 system prompt 写死 XML 格式、客户端正则解析——这是为自训练模型 MiroThinker（基于 Qwen 后训练）做的协议 co-design，对 GPT/Claude 等通用模型不友好；全工具依托 MCP server（仅 6 类十来个工具 vs Claude Code 40 多个），走"少而重"路线（一次 google_search 拉 10 条结构化结果、scrape_website 用 Jina Reader 转整页 markdown）。agent loop 前后各有独特处理：开头用强模型分析题面陷阱（把"理解题"与"解题"解耦，题面误读是 deep research 最廉价又最致命的失败模式），结尾用两次 LLM 调用把总结编译成评测格式（先判 number/date/time/string 类型再套格式 prompt）。context 管理采用"仅保留最近工具结果 + 占位符替换"（默认关闭 keep_tool_result=-1，为 32K 小模型服务）和 summary 阶段 retry-and-rollback（优先删最近的死循环挣扎 turn）；sub agent 模式下 main agent 仅保留 reasoning 工具，context 增长按子任务数而非工具调用数线性 ^[raw/articles/2026-06-04-Deep-Research-Harness-的独特设计-炼钢AI.md]

## 关键要点

- 工具调用本质是 prompt-based：所谓结构化 dict 会被 provider 序列化进 chat template，模型输出的特殊 token 字符串由服务端解析——MiroFlow 干脆绕开原生协议，在用户控制的 system prompt 里写死 XML 格式。
- 六类 MCP 工具：searching（google_search/wiki/存档网页/scrape）、reading（markitdown 转 markdown）、code（E2B 沙箱 + 长存活 Jupyter Kernel，变量跨轮次存续）、image-video、audio（Whisper 转写）、reasoning（外包给 o3/Claude extended thinking）。
- search_archived_webpage 基于 Wayback Machine 查历史快照——查"2018 年 Q3 财报数字"这类已下架内容的关键能力，Claude Code 没有。
- 评测公平性问题：MiroFlow 论文中 "MiroThinker 打过 Claude" 严格说只证明"自家训练 + 框架联合设计在自家协议下胜出"，公平比较需给每个模型配其最擅长的脚手架协议（co-design 系统）。
- 工程哲学：给 main agent loop model 减负——格式约束、题面分析、最终答案编译都剥离给外部 LLM 调用兜底，让小模型只负责"工具调用 + 信息整合"。

## 来源

- 原文: [[raw/articles/2026-06-04-Deep-Research-Harness-的独特设计-炼钢AI.md|1.前言]]
