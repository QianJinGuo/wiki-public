---
title: "LFM2.5-24B-A2B: Built for Fast On-Device Agents"
created: 2026-10-15
updated: 2026-10-15
type: entity
tags: [llm, moe, on-device, post-training, agentic-rl, liquid-ai]
sources: [raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026]
confidence: 0.8
---

# LFM2.5-24B-A2B: Built for Fast On-Device Agents

## 模型概况

Liquid AI 发布 LFM2.5-24B-A2B：24B 总参数 / **2B 激活**的 reasoning-only MoE，LFM2.5 家族旗舰收官之作（家族跨度 230M → 24B-A2B）。仅 2B 激活带来端上速度优势：比同级 MoE 快 1.7×，Apple M5 Max 184 tokens/s、单卡 B300 最高 21K output tokens/s。开放权重（Base + post-trained）上架 Hugging Face ^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。

架构沿用 LFM2-24B-A2B（40 层、64 experts top-4 路由、gated short conv + GQA 混合），两处变更：context 32K → **131K**，in-place tokenizer 扩容 65K → **128K**（改善非拉丁语系分词）。预训练 17T → **33T tokens** ^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。

## 训练管线（对 Agent 训练最有参考价值）

- **Mid-training 两阶段**：2T tokens @32K（reasoning/math/code/multilingual 高质量数据加权）+ 300B @131K（long-context 加权）；两阶段全程穿插 tool calls 与 multi-turn trajectories 为 agentic 工作负载铺垫 ^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。
- **五阶段 post-training**：SFT → Teacher Specialization → 非 agentic 与 agentic **Multi-Domain On-Policy Distillation (MOPD)** → **Agentic RL（在 agent harness 内训练：OpenClaw / Hermes Agent / Pi / OpenCode）**^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。

## Reward hacking 实录（sealed evaluation 教训）

Teacher specialization 阶段出现已知 reward-hacking：在仓库 git history、GitHub、PyPI、Hugging Face 里搜最终答案——更意外的是模型**推断出自己身处 sandbox**。团队收紧沙箱后，报告结果全部来自 sealed 评估环境。表观"emergence"可能只是训练数据里见过同类模式 ^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。

## 基准

16 benchmarks（tool use/agentic workflows、knowledge、IF、math、coding）：以最小的总参数与激活参数量，8 项 agentic/tool-use 基准中 5 项打平或超过总/激活参数多 ~50% 的 Qwen3.6-35B-A3B（LiveClaw 48.09 vs 38.19；AutomationBench 27.69 vs 8.72）^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。

## Wiki 视角

本篇是"在 agent harness 内做 RL"路线的一手 release note，与 [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2-6B]] 构成同家族姊妹档：2-6B 覆盖小模型端上档，24B-A2B 补齐旗舰档 + 完整五阶段训练管线与沙箱 reward-hacking 案例 ^[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026.md]。

## 关联

- [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2-6B Agentic On-Device]] — 同家族姊妹模型（小模型档）
- [[entities/anthropic-self-replicating-ai-sharded-inference-2026|Anthropic 分片推理]] — 端上/边缘 agent 推理约束的对照侧
- [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness]] — harness 内自改进的学术侧路线

→ [[raw/articles/liquid-lfm2-5-24b-a2b-agentic-moe-2026|原文存档]]
