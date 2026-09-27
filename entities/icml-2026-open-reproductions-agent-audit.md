---
title: "ICML 2026 Open Reproductions — 大规模 Agent 驱动的论文复现审计"
created: 2026-08-16
updated: 2026-09-27
type: entity
tags: [agent, evaluation, reproducibility, icml, coding-agent, scientific-research, harness]
sources: [raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# ICML 2026 Open Reproductions — 大规模 Agent 驱动的论文复现审计

Hugging Face + alphaXiv 于 2026-07-15 ~ 08-02 举办的 ICML 2026 Open Reproductions challenge——社区成员自带 coding agent 逐 claim 复现 ICML 2026 论文，最终以 **2,226 篇论文（占全会 34%）被尝试、35,908 个 claim 被裁决** 的规模，成为迄今最大规模的开放 ML 会议逐 claim 审计。^[raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml.md]

## 挑战机制

- **Claim 抽取**：主办方索引全部 6,341 篇 ICML 2026 论文，从摘要中提取核心科学 claim，让 agent 从「可检验的具体目标」而非 40 页 PDF 起步
- **自带 Agent**：参与者使用 Claude Code / Codex / Cursor / OpenResearch `orx` 等任意 agent 框架，主办方提供单命令拉取论文 + claims + 指令的接口
- **Trackio Logbook**：每次复现产出静态 HF Space logbook（复现报告 + 运行代码 + artifacts + 可选完整 agent 执行轨迹），审计过程本身可审计
- **自动裁决**：Logbook Judge 使用开源权重模型 GLM-5.2 重读每份 logbook，逐 claim 给出 verdict：`verified` / `falsified` / `toy`（降规模证据）/ `inconclusive`，且明确要求不信任 logbook 的自我评估
- **规模**：1,221 名成员、6,816 份 logbook、2,962 个 HF Jobs 云任务（$20 免费算力）、274 份完整 agent 轨迹数据集公开

## 关键发现

- **51% 的已检论文（1,103 篇）至少 1 个 claim 被独立验证**：其中 266 篇完全复现（所有 claim verified），632 篇部分复现且无 falsified；共 3,978 个 claim 被真实实验确认
- **23% 的已检论文（496 篇）至少 1 个 claim 被 falsified 或争议**：49 篇全部 claim 被推翻，242 篇独立复现团队对同一 claim 得出相反 verdict——「可复现性不是二元的，它是对抗性的」
- 502 篇仅有 toy-scale 证据，280 篇无法判定（缺失 artifacts 最常见）

## 代表性 falsification 案例

| 论文 | 声称 | 审计发现 |
|------|------|---------|
| Towards Optimal Robustness in Learning-Augmented Paging | 鲁棒性 H_k + O(1) | 实测 additive term 按 0.38·ln k 增长，re-implementation 在 k=1024 确认 ~9 sigma；真实为 H_k + Θ(log k) |
| Attention's forward pass and Frank-Wolfe | token particles 坍缩到原点 | 三个独立团队找到反例（t=224 / ~3,800 / 6,416 步出现违例）——有限时域检查过早停止导致他人"验证成功" |
| Self-Distillation Enables Continual Learning | 中心方程/理论基于 reverse KL | 代码默认（产出全部结果的）计算 forward KL；+4pp headline 结果无法复现，作者已上传 arXiv 澄清版 |
| Do Transformers Need Three Projections? | 3.1% quality cost for 50% cache reduction | ~66% 被评 label 位置是 EOS padding token（趋近零 loss），perplexity 被低估约 3 倍；修正后约 9.4% |

**假 falsification 也存在**：一份 logbook 声称"方法比 baseline 慢 2x"，实为复现方算术错误（per-trajectory 时间对比 per-batch-of-50 时间），正确归一化后数据反而确认了论文声称的 8x 加速。

## 人类在 Agent 科研中的角色

- **纯 agent 执行有真实边界**：agent 陷入局部循环、误读 scale-dependent 行为（paging 论文多个"verified"来自 log-k 增长显现前的检查）、在 units mismatch 上构建整个 falsification
- **部分评估不可约地需要人类**：human-in-the-loop 冠军案例——量化鲁棒性论文的数值指标显示"无坍缩"，但图像是否可用是感知问题；参与者构建 review UI 亲自裁决全部 128 对图像
- **人类角色 = 有效管理智能**：类似 PI 为研究生搭建环境（算力、harness、数据、适时反馈），收益最大的参与者是「搭建正确环境 + 提出正确问题」的人

## 行业意义

这是 agent 在科研评估中从"辅助写作/跑实验"走向"独立 claim 验证基础设施"的实证：agent 复现规模（2,226 篇/19 天）远超人类审稿容量（审稿人自述"未仔细检查 proofs"的 spotlight 论文被复现推翻）。审计产物（logbook + 轨迹 + 裁决）全部公开，主办方希望记录被打破。

## 深度分析

### 复现性是对抗过程，不是二元判定

23% 的已检论文至少 1 个 claim 被 falsified，其中最值得注意的是 242 篇出现独立复现团队对同一 claim 得出相反 verdict 的情形——同一实验设置，不同 agent 得出不同结论，说明任何单次复现都不是终审。Frank-Wolfe 定理的反例最早在 t=224 步才出现违例，此前所有"verified"都是检查时域不足的产物；反过来，也有 logbook 在 units mismatch（per-trajectory 对比 per-batch-of-50）上构建出假 falsification。可信度不取决于"是否被复现过"，而取决于检查的时域长度、数值边界与对抗强度。^[raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml.md:61-65]

### 规模驱动的审稿危机与 agent 的对称解法

ICML 2026 收到 23,918 篇投稿、录取 6,352 篇，约为前一年两倍——增长部分正由 agent 加速实验与写作驱动；而志愿者审稿容量没有同步扩张，一篇 spotlight 论文的审稿人自述"未仔细检查 proofs"，该论文随后被复现推翻。结构性洞见在于对称性：造成投稿洪水的同一技术（coding agent 写论文）恰好也是解法（coding agent 逐 claim 读论文并验证），把"一位审稿人一个周末"的检查压缩成"数千次并行、每次一个下午"。^[raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml.md:20-32]

### 有限时域检查是系统性盲区

多个案例共享同一失效模式：检查在效应显现之前停止。paging 论文多个"verified" verdict 来自 log-k 增长可见之前的检查，只有把 sweep 扩展到 k=1024 才以约 9 sigma 确认 Θ(log k) 的真实增长；"Do Transformers Need Three Projections?" 中约 66% 的被评 label 位置是趋近零 loss 的 EOS padding token，perplexity 被稀释约 3 倍。默认超参与有限时域会把真问题标记为"验证通过"，最干净的反例甚至用精确有理数算术消除了浮点歧义的存在空间。^[raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml.md:76-88]

### 人类价值从执行移向问题定义

agent 最可靠的结果来自人类 steering 的 workflow：重新指向 agent、质疑假设、在烧掉一周算力之前判定实验前提错误。human-in-the-loop 冠军案例进一步显示分工形态——量化论文的数值指标显示"无坍缩"，但图像是否实际可用是感知问题，agent 自建 review UI、人类亲自裁决全部 128 对图像、agent 事后验证标注一致性。人类角色被重新定位为 PI 式的"有效管理智能"：搭建 harness、算力与数据环境，提出正确的问题，然后让 agent 跑。^[raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml.md:103-109]

## 实践启示

1. **把审计设计成 claim 级而非论文级**：主办方预先抽取核心科学 claim，让 agent 从可检验的具体目标起步——35,908 个 claim 级裁决远比 2,226 个论文级模糊判断更可操作、更可聚合
2. **不信任被评估方的自我评估**：Logbook Judge 被明确要求不信任 logbook 自评；同时让审计过程本身可审计（公开 logbook、代码、artifacts、完整 agent trace），形成双层验证
3. **对 claimed falsification 做 adversarial re-verification**：35 个正式 falsification 声称全部经过重读论文、重推导数学、从论文文本重新实现后才确认——这既筛出了真问题，也拦截了复现方自身的算术错误
4. **把数值检查 sweep 到失效边界**：扩展 k、延长 t、修正评估位置的归一化——默认参数下的"验证通过"可能是检查窗口太短的假象；必要时用精确有理数算术排除浮点歧义
5. **在感知类判断上显式引入 human-in-the-loop**：数值指标无法回答的问题（图像可用性、生成质量），采用「agent 搭建评审工具 → 人类裁决 → agent 验证标注一致性」的循环，而不是让 agent 对感知结论下终审
6. **以作者确认为审计闭环的终点**：用 "here is what we found, here is all the evidence, do you agree?" 的框架联系作者，早期响应积极，多个 arXiv 修正已在途——审计的产出是修正，不是定罪

## 相关实体

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]] — 本挑战本质是 claim 级评估基础设施，与基准框架互补
- Agent 评估基准 — 复现审计可作为科研产出评估的新基准形态
- [[concepts/agent-orchestration-patterns|Agent 编排模式]] — BYO agent + 统一接口的社区协同模式
- [[entities/agent-tools-research|Agent 工具研究]] — 工具与 agent 能力的实证关系
- [[entities/adopting-ai-coding-agents-six-lessons|采用 AI coding agent 的六个教训]] — coding agent 实际能力边界的并行证据
- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 框架实践]] — 长时域 agent 行为的另一视角
- [[concepts/verifier-driven-development|Verifier-Driven Development]] — 自动化验证驱动的开发范式延伸

→ [[raw/articles/what-we-learned-by-reproducing-2200-papers-from-icml|原文存档]]
