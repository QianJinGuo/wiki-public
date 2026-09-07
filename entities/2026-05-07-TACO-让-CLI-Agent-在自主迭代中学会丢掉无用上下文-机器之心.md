---
title: "TACO 让 CLI Agent 在自主迭代中学会丢掉无用上下文 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-07-TACO-让-CLI-Agent-在自主迭代中学会丢掉无用上下文-机器之心]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-05-07-TACO-让-CLI-Agent-在自主迭代中学会丢掉无用上下文-机器之心.md|原文存档]]

sha256: dd5f1d21f23c967a6c284b4213548c1f4ad4f8031fca8e247fcf6cbe2cabad15 ^[raw/articles/2026-05-07-TACO-让-CLI-Agent-在自主迭代中学会丢掉无用上下文-机器之心.md]

## 摘要

机器之心介绍曼彻斯特大学、北航、港科大与 MAP 团队提出的 TACO（Terminal Agent Compression）——一个无需训练、即插即用的终端智能体自进化观测压缩框架。其出发点是：长程 CLI Agent 的瓶颈不是上下文窗口不够大，而是上下文在多轮交互中越来越"脏"——在 TerminalBench 2.0 轨迹中，Qwen3-Coder-480B、DeepSeek-V3.2、MiniMax-M2.5 的 raw prompt 有 24.6%–44.1% 可被人工判定为低价值冗余。TACO 用轻量级自进化规则引擎（触发条件 + 保留/剔除模式组成的函数）替代人工预设截断和 LLM 实时总结，通过任务内动态纠偏、全局跨域沉淀三个阶段（Terminal Output Compression、Intra-Task Rule Set Evolution、Global Rule Pool Evolution）持续学习哪些输出可安全过滤、哪些行动线索必须保留 ^[raw/articles/2026-05-07-TACO-让-CLI-Agent-在自主迭代中学会丢掉无用上下文-机器之心.md]

实验显示 TACO 在 TerminalBench 1.0/2.0、SWE-Bench Lite、CompileBench、DevEval、CRUST-Bench 上同时提升任务成功率和 token 效率；相同 token budget 下在六个模型上准确率均更高，说明提升来自有效信息密度而非更多交互。案例中 10,071 字符的 apt-get 安装日志被压到 73 字符，而 SQLite 编译输出中 -fprofile-arcs、-ftest-coverage 等覆盖率编译参数被保留。收敛判断不用测试集（避免评测泄露），而用 Global Rule Pool Top-30 规则的 Retention——三个模型多轮演化后超过 90%，且规则池稳定与性能稳定同步出现 ^[raw/articles/2026-05-07-TACO-让-CLI-Agent-在自主迭代中学会丢掉无用上下文-机器之心.md]

## 关键要点

- TACO = Terminal Agent Compression，arXiv:2604.19572，代码开源在 github.com/multimodal-art-projection/TACO；共同一作 Jincheng Ren、Siwei Wu、Yizhi Li。
- 静态压缩对比实验（Qwen3-Coder-480B）：LLM Summarize token 成本最低但准确率明显下降；TACO token 成本不是最低，却取得最高准确率和最小方差。
- 三个自进化阶段：按 active rules 压缩输出（错误/异常信号保守处理）→ 任务内生成新规则并用 over-compression signal 纠偏 → 有效规则写回 Global Rule Pool 供后续任务检索复用。
- 压缩行为示例：10,071 字符 r-base 安装日志 → 73 字符；objdump 输出过滤重复 hex dump 但保留 call 指令、符号标签和关键地址。
- 与 Terminus-2 插槽集成后在多种强模型上稳定提升；CompileBench 上准确率不变但 token 消耗明显下降。

## 来源

- 原文: [[raw/articles/2026-05-07-TACO-让-CLI-Agent-在自主迭代中学会丢掉无用上下文-机器之心.md|TACO 让 CLI Agent 在自主迭代中学会丢掉无用上下文 机器之心]]
