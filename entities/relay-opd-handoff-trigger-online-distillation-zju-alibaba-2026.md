---
title: "Relay-OPD：在线蒸馏的前缀失败纠偏（浙大×阿里，2026）"
created: 2026-08-25
updated: 2026-08-25
type: entity
tags: [post-training, on-policy-distillation, distillation, math-reasoning, speculative-decoding, zju, alibaba]
sources: [raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026]
confidence: 0.78
provenance_state: extracted
---

# Relay-OPD：在线蒸馏的前缀失败纠偏（浙大×阿里，2026）

> **一句话**：在线蒸馏（OPD）中，学生推理一旦早早走偏，教师也会被错误前缀带跑。Relay-OPD 在生成过程中在线检测"跑偏"位置（交接触发点），让教师像接力赛一样短暂"接棒"纠偏再交还学生，八个数学基准全最优/次优，训练轨迹长度削减过半。^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

## 摘要

浙江大学生物信息学（浙大）联合阿里巴巴提出 Relay-OPD（Relay On-Policy Distillation）。核心洞察：在线蒸馏让学生在自己生成的轨迹上接受教师逐 token 的稠密监督，但学生自生成轨迹必然带着自己的错误——长链推理中一旦开头选错方向，后续所有生成都建立在该偏差之上，教师监督越来越不可靠，训练算力被浪费在偏离正轨的长续写上。Relay-OPD 在线检测推理"跑偏"位置，让教师短暂接棒纠偏再交还学生。^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

## 核心要点

- **前缀失败（Prefix Failure）**：学生推理早期选定错误方向后，自回归生成让后续内容都在错误前提下展开，滚雪球形成冗长失败轨迹 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **关键观察**：在走偏前缀上，教师与学生"续写本能"截然不同——教师倾向停下来反思（下个 token 大概率是 Wait/But/However），学生倾向沿原方向写下去。实测某位置教师 74.4% 想说 "But"，学生 50.6% 想接 "So" ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **交接触发点（handoff trigger）**：教师在当前前缀上最偏好的下个 token 属于反思词表 R，而学生 top-K 候选中不含反思词时判定触发（K=5），无需 verifier/过程标签/奖励模型 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **效果**：八个数学基准全最优/次优，1.7B 学生较标准 OPD 平均提升 +5.73%，训练轨迹长度削减 50.7% ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

## 深度分析

### 1. 问题与现有方案的局限

已有补救各有结构性局限：固定长度截断（ESR、FastOPD）一刀切，与推理实际失败位置无关；离线重写（TRD）干预太晚且重写痕迹明显；token 级混合（SKD）无"推理方向已失败"的显式信号。缺的是既在线发生、又由推理状态本身决定干预位置的机制。^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

### 2. 轨迹干预实验的三个结论

- **纠偏可极其局部**：只在每个触发点替换那一个反思 token（教师 token 仅占全部 0.35%），准确率从 27.73 提至 34.96（+7.23%）^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **干预价值高度前置**：保持干预长度不变，把位置从最早触发点推后，准确率从 41.99 跌至 33.98 再至 29.49——前缀变长后教师被学生上下文带跑，师生差距收窄，晚到接管无力回天 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **延长干预收益快速饱和**：单次接管从 3 段延至 6 段，教师 token 占比从 17.52% 升至 28.52%，准确率却停留 41~44 区间。干预应"点到为止"——既要趁早又克制 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

### 3. Relay-OPD 三个关键设计

- **免标签交接触发器**：预定义反思词表 R（Wait/But/However 及大小写、前导空格变体）。教师最偏好的下个 token 属于 R 而学生 top-K 不含任何反思词时触发 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **接力预算 (M, L)**：整条轨迹最多 M 次教师接管；每次以触发反思 token 起笔，继续生成 L 个自然段（以 \\n\\n 为界，平均每段约 23.2 token）——按"段落"而非固定 token 数计量，保证接管收尾于结构完整推理单元。主实验取 (M,L)=(2,3) ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- **面向接力轨迹的训练目标**：教师给接力轨迹的指导并不等于其理想发挥（教师也被学生前缀牵着走）。学生不照单全收，直接在实际生成的接力轨迹上做 single-sample 蒸馏，有选择吸收教师纠偏信号 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

### 4. 工程实现：投机解码统一引擎

交接触发器要求每个位置拿到教师对下个 token 的分布判定，且教师与学生同轨迹反复交替生成。团队将整个接力过程统一进投机解码引擎：学生为 draft model、教师为 target model。学生段中 target 即学生自身，draft 全接受；教师接管段为标准投机拒绝采样。教师本就要对学生草稿做批量验证，验证中算出的教师 logits 顺带给出触发判据，持续监测教师意图零额外开销；投机采样正确性保证单引擎产出与"两模型真实轮流生成"分布完全一致。^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

### 5. 实验结果

以 Qwen3-4B-Instruct-2507 为教师、Qwen3-0.6B/1.7B（Non-Thinking）为学生，DAPO-Math-17K 英文子集上基于 verl + vLLM 训练，全程无 verifier/过程监督/答案正确性标签，在 AIME 2024/2025/2026、MATH500、AMC 2023、OlympiadBench、HMMT 2026 Feb/Nov 八个基准评测 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

- 1.7B 学生平均准确率 46.96，较标准 OPD（41.23）提升 +5.73%；AIME 2025/2026 分别 +7.29%/+7.19%；超过最强轨迹干预基线 FastOPD（45.47）+1.49% ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- 效率：1.7B 平均训练轨迹长度 2,296 token，较 OPD 的 4,658 削减 50.7%（0.6B 削减 63.9%），仅 35 步达最优 checkpoint（OPD 55 步）^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- 推理更短答得更准：vs FastOPD，AIME 2025/2026/HMMT Feb 2026 回复长度缩短 17.9%/14.2%/28.3%，准确率反升 +2.39%/+4.17%/+1.14% ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- 教师介入强度自适应衰减：用满接力预算轨迹占比从初期 75%~85% 降至 50%~60%，教师 token 占比从约 13% 快速回落，约 20 步后稳定 2%~3% ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]
- 消融：限定 M=1，"触发即停"仅 43.48，补 L=3 教师接管段后升至 46.25（+2.77%）——收益来自修正上下文与局部推理示范，不只动态截断 ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

### 6. 团队与资源

第一作者徐皓雷（浙大计算机博士生二年级，大模型推理/后训练/可解释性），共同一作 & Project Lead 洪海文（阿里安全 AGI 实验室-御风大模型团队，大模型预训练/多模态/Self Play），通讯作者鲁伟明教授（浙大）。论文 arXiv:2607.26057，项目 zju-real.github.io/Relay-OPD，代码 github.com/ZJU-REAL/Relay-OPD ^[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026.md]

## 关联实体

- [[concepts/model-distillation-compression|模型蒸馏与压缩]]
- [[concepts/speculative-decoding|投机解码]]
- [[entities/on-policy-distillation-vs-offline-distillation-loster|在线 vs 离线蒸馏]]
- [[entities/xopd-on-policy-distillation-landscape-banana-2026|XOPD 在线策略蒸馏全景]]
- [[entities/d-opsd-diffusion-llm-on-policy-self-distillation|D-OPsD 扩散在线自蒸馏]]
- [[entities/opd-revisiting-failure-modes-simple-fixes-storm|OPD 失败模式重访]]
- [[entities/seed-self-evolving-opd-long-horizon-agent-rl-tsinghua-zju-2026|自演化 OPD 长程 Agent RL]]

## 原文存档

→ [[raw/articles/relay-opd-handoff-trigger-online-distillation-zju-alibaba-2026|原文存档]]
