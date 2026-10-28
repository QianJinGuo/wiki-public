---
title: "Jeva：NUS 把决策原则做进架构的 System One 决策模型"
created: 2026-10-26
updated: 2026-10-26
type: entity
tags: [decision-model, system-one, architecture, agent, evaluation, model, research]
sources: [raw/articles/jeva-decision-model-architecture-nus-2026]
confidence: 0.85
provenance_state: extracted
---

# Jeva：NUS 把决策原则做进架构的 System One 决策模型

新加坡国立大学 xML Lab 的 **Jeva** 把"决策"本身当作架构设计的起点：模型不生成推理长文，而是直接输出选择、概率或分数（Choice / Noul / Score 三种输出），并把四项决策结构特性——答案缺席识别、顺序不变性、候选可扩展、计算可复用——做成架构的内在能力。核心主张一句话：**Decision principles, built into the architecture.**^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

论文 arXiv:2610.18773，项目页 liqiiiii.github.io/Jeva/，代码 github.com/LiQiiiii/Jeva，权重 huggingface.co/LIQIIIII/Jeva-4B；Jeva-4B 使用 Qwen3.5-4B 骨干加一个决策块。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

## 四项结构特性：不是功能清单，而是同一问题的四个侧面

**Null-aware（没有合适答案也是一种答案）**：把"是否存在可接受候选"与"候选之间谁更好"分开判断。退款场景里候选只有"修改地址/查询物流/重置密码"时，即使某个分数最高也不代表能解决问题，Jeva 为此提供明确的 null 输出，而不是强行选一个。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

**Permutation-invariant（换顺序不换判断）**：同一组候选从 A、B、C 改为 C、A、B 不应改变含义。Jeva 按候选集合对称聚合、以同一规则打分，让概率跟随候选身份而非列表位置——关心的不是模型是否仍选"第二项"，而是是否仍选同一个答案。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

**Option-scalable（选项变多，决策方式不变）**：候选可以是十几个标签，也可以是上千份文档。Jeva 对每个候选应用共享评分规则，无需为新增候选扩展固定答案符号；在大候选实验中直接处理完整集合，而不靠多轮淘汰逐步缩小范围。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

**Cacheable（复用选项计算，不复用旧答案）**：文档、政策条款、评分标准往往比用户问题稳定。Jeva 可保留这些选项的计算结果，在问题变化、候选重新组合时继续使用，而当前请求的判断仍重新执行——缓存的是可复用信息，不是上一次的结论。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

## 架构：把"理解文本"与"作出当前判断"分开

Jeva 用共享编码器分别表示请求和选项，再由轻量决策模块读取整个候选集合、只更新请求状态：选项表征保持可复用，判断随当前问题与候选内容更新。分别编码不等于独立打分——固定初始请求状态与候选 A、B，仅把第三个候选从 C₊ 换成 C₋，就能让更新后的状态改变方向、使偏好从 A 转向 B。这证明单个决策块已能表达集合相关偏好，而只依赖"请求 × 单候选"的独立评分无法因第三个候选变化而反转 A、B 的相对排序。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

输出层把两件事再拆开：用 **q** 描述候选之间的相对偏好，用覆盖概率 **ρ** 估计是否存在合适答案，再合成为包含 null 的完整分布 **p**。同幅降低两个候选分数时相对概率仍是 73%/27%，但覆盖概率会从 88% 掉到 12%，null 概率随之升高；固定已有分数加入一个低分候选时，覆盖概率不变，而候选间概率重新分配。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

## 实验：四个结构问题各有对应证据

**答案缺席识别**：BANKING77 上构造 3,080 个请求、每个 16 个候选，其中 588 个答案缺席。Jeva 的 null 检测 AUROC 达 **98.66%**，高于 KEV-BF16（93.61%）、InternDecision-4B（84.27%）与 JevK5（79.44%）；在把约 5% 有答案请求误判为无答案的阈值下，能识别约 94% 的缺席请求。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

**顺序不变性**：600 个 Choice 样本上测试原始顺序、倒序与最多三种随机排列，按候选身份对齐结果后 Jeva **没有出现决策翻转**，原始准确率与"每种顺序都答对"的准确率均为 76.67%；对照的 Jev-1.13.0、InternDecision-4B、JevK5 翻转率分别为 17.00%、22.67%、26.17%。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

**候选扩展**：ToolRet 诊断覆盖 32 到 2,048 个候选的六组设置。2,048 候选下 Jeva 准确率 69.88%、单张 H200 请求中位延迟 12.04 秒；采用递归选择的 JevK5 为 67.47%、约 28.35 秒——同设置下同时取得更高准确率与约 **2.35 倍**速度。更严格的检验固定 83 个问题、把候选集从 32 逐步扩到 2,048：Jeva 在所有规模下答对其中 **58 个**，JevK5 为 48，CLM 为 7。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

**计算复用**：MLDR 长文档实验中，面对新组合但全部已缓存的候选集，请求中位延迟从约 2.10 秒降至 **75.1 毫秒**（27.98 倍加速），且不同缓存条件下未观察到决策改变。另有一次 ABC → BCD → ACD 的演示：单选项缓存命中依次为 0/3、2/3、3/3，每次仍重新编码请求并重算当前判断。^[raw/articles/jeva-decision-model-architecture-nus-2026.md]

## 与其他决策模型路线的关系

这条路线与 UIUC 的 JevAgentHarness 构成姊妹面：后者关心"决策模型如何在智能体循环里与 LLM 共享控制权"——把循环拆成 8 个决策点、5 种集成方式（Replace / Cascade / Assist / Guard / Trigger），并用阈值委托在把握不足时交还 LLM，实测 4B 决策模型让客服与家居智能体成本降低 81.1%–89.0%、提速最高 2.5×、成功率损失不到 3 个点。Jeva 补的是"决策模型本身该怎么设计"这一层：候选集合参与判断、缺席可表达、顺序无关、选项可缓存。^[raw/articles/jeva-decision-model-architecture-nus-2026.md, raw/articles/jevagentharness-decision-model-shared-control-uiuc-2026-10-18.md]

## 可迁移的工程启示

1. **把"是否该回答"作为独立输出维度**。大量 agent 失败不是选错，而是在没有合适选项时强行选一个；把覆盖概率 ρ 与相对偏好 q 分开建模，才能给出可校准的拒答信号。
2. **顺序不变性要靠聚合方式保证，不能靠数据增强**。按候选身份对称聚合、共享打分规则，是从架构上消除位置偏置；靠训练时随机打乱只是降低概率，不消除翻转。
3. **候选规模扩展的关键是"一次决策头看完整集合"**。递归缩小范围（多轮小型选择）在大候选集上既慢又容易错，Jeva 在 2,048 候选下同时赢准确率与延迟，说明集合级比较与分批编码可以解耦。
4. **缓存"选项理解"而非"决策结论"**。选项语义稳定、判断随请求变化，这条切分让长文档/政策条款类 RAG 式决策拿到近 28 倍延迟收益，同时不引入陈旧结论风险。

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] — null 检测 AUROC、翻转率、候选扩展准确率是一组可直接复用的决策质量指标
- [[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]] — 决策模型作为 harness 里"廉价决策层"的组件角色
- [[entities/liquid-ai-d1-edge-decision-models|Open d1：边缘决策模型]] — 同为"单次前向输出决策而非逐 token 生成"的 System One 路线，端侧延迟导向
- [[entities/gka-variable-test-time-compute-2026|GKA：可变测试时计算]] — 决策成本与推理预算的另一侧对照
- `raw/articles/jevagentharness-decision-model-shared-control-uiuc-2026-10-18` — 姊妹论文：决策模型与 LLM 共享控制权（raw 存档，实体待建）

→ [[raw/articles/jeva-decision-model-architecture-nus-2026|原文存档]]
