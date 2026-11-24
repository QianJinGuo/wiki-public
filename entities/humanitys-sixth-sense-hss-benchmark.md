---
title: "Humanity's Sixth Sense (HSS) — 直觉视觉推理基准"
created: 2026-10-08
updated: 2026-11-23
type: entity
tags: [benchmark, multimodal, vision, evaluation, mllm]
sources: [raw/articles/humanitys-sixth-sense-hss-benchmark]
confidence: 0.7
provenance_state: extracted
---

# Humanity's Sixth Sense (HSS) — 直觉视觉推理基准

> **Background**：本文档基于 Scale AI（labs.scale.com）发布的 HSS 基准论文摘要页建立。因全文（arXiv/PDF）在抓取时不可达（Cloudflare 5xx + Jina 代理窗口不可达），仅存档摘要页内容。

## 核心定义

HSS（Humanity's Sixth Sense）是一个针对**直觉视觉推理**（intuitive visual reasoning）的多模态基准。现有视觉基准要么针对刻意的专家级分析，要么针对低层感知，而人类一眼完成的直觉推理（隐含的时间、空间、社会、抽象结构推断）长期缺测。HSS 覆盖图像与视频输入，采用结构化分类法组织条目，每条配有人类撰写的 prompt，探测人们一眼就能推断的隐含结构。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]

## 关键数据

- **人类 vs 前沿模型鸿沟**：人类参与者准确率 93.1%，最强模型 GPT-6-astra 在最大推理力度下仅 53.6% — 差距约 40 个百分点。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]
- **任务示例**：一眼捕捉过去成因与未来轨迹；快速一瞥判断车辆能否停进两车之间；几秒视频判断房间内谁有权威。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]
- **Agentic 设置**：对 HSS 应用动态视觉操纵（dynamic visual manipulation）的 agentic 设置可缩小但不能闭合鸿沟。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]

## 意义

HSS 将直觉视觉推理确立为可度量的能力轴，把注意力引向当前基准 scaling 尚未覆盖的能力面 — 与现有复杂任务上的强表现形成反差，说明感知-知识型 benchmark 饱和与直觉推理缺口之间存在结构性错位。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]

## 深度分析

### 直觉推理为何被称作"第六感"：基准的问题意识

HSS 的问题意识来自一个被长期忽略的空白带：现有视觉基准要么考核刻意的、专家级的深度分析，要么停留在低层感知，而人类在"一眼之间"完成的推理 — 从单帧读出事件的过去成因与未来轨迹、扫一眼判断车能否停进两个车位之间、几秒视频里识别出房间里谁说了算 — 从来没有被系统度量过。HSS 的设计思路是把这种快直觉当作独立能力轴：用结构化分类法组织条目，覆盖图像与视频两种输入，每条配人类撰写的 prompt，专门探测隐含的时间、空间、社会与抽象结构。这与 [[entities/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly]] 的视觉因果探测思路同属一个方向：考的不是"看到了什么像素"，而是"从画面中推断出了什么隐含结构"。

任务样例本身就说明了这种探测的广度：过去成因与未来轨迹考的是时间因果，停车位判断考的是空间量化，"谁有权威"考的是社会结构。这四个维度（时间、空间、社会、抽象）在传统基准里各有零散覆盖，但把它们统一进"一眼可判"的难度档位、并按结构化分类法组织成可比的题集，是 HSS 的方法论贡献。这也与 [[entities/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly]] 的视觉因果探测思路同属一个方向：考的不是"看到了什么像素"，而是"从画面中推断出了什么隐含结构"。

### 40 个百分点的鸿沟说明了什么

人类参与者 93.1%，最强模型 GPT-6-astra 即便开到最大推理力度也只有 53.6%，差距约 40 个百分点。值得注意的不是差距本身，而是它的形状：这些模型在需要高级感知与知识的复杂任务上表现很强，却在人类直觉层面大幅失手 — 说明缺口不在感知分辨率或知识覆盖，而在对隐含结构的快速推断能力上。推理力度这个旋钮在这里几乎失效，这与 [[entities/noam-brown-ai-evaluation-reasoning-budget-performance-cost-curve]] 中"推理预算—性能曲线因能力轴而异"的观察一致：有些能力靠加大 thinking 换不来，直觉视觉推理正是典型样本。

### Agentic 视觉操纵：缩小但不能闭合

论文进一步尝试把 HSS 放进 agentic 设置，允许模型对输入做动态视觉操纵（dynamic visual manipulation），结果是鸿沟缩小但未闭合。这传递了两层信息：其一，给模型"动手看"的行动空间确实能买到部分增益，纯静态前馈式 perception 不是唯一路径；其二，剩余差距说明模型缺的不只是验证手段，而是把视觉观察与行动假设串起来的世界动态模型 — 这正是 [[entities/world-model-evaluation-position-paper-nju-2026]] 所主张的评估议程：时空因果一致性应当成为独立考核维度，HSS 可以视为它在"人类直觉难度"上的一个切片。

### 与评测方法论的关系：人类直觉作为天花板

HSS 把人类准确率（93.1%）放在了天花板位置，模型分数向下对标 — 这种"人类直觉即上限"的评测设计，与 [[concepts/verifier-paradox]] 讨论的验证不对称问题形成互补：直觉任务上人类验证近乎零成本（一眼即判），机器生成本身却远未达标。同时它也印证了 [[concepts/eval-surface-rotation]] 的判断：当感知—知识型基准陆续饱和、分数失去区分度时，真正有信息量的新评测面来自 scaling 尚未覆盖的能力轴，而非把现有考题加难。

## 实践启示

1. **评估多模态系统时，加一组"一眼可判"的直觉题做探针。** 专家级分析与低层感知基准都过了，不代表系统能完成人类水平的快速结构推断；HSS 式的直觉题是独立能力轴，不能被现有高分外推。
2. **先判断失败类型，再调推理力度。** 直觉视觉推理上最大推理力度只有 53.6%，说明这类失败是"结构型"而非"算力型"；盲目加大 thinking 预算对这类能力轴收益接近于零，应转向数据与任务形态的补强。
3. **把 agentic 视觉操纵当作部分解而非终解。** 允许模型动态操纵输入能缩小差距但不能闭合，规划视觉 agent 能力栈时应预期"行动化感知"只覆盖一部分直觉缺口，剩余部分要靠世界模型式的时序因果建模。 对评估设计者的反向启示是：在静态基准失手的模型，值得先测一遍 agentic 版本再下结论 — 部分失分可能源于缺少主动获取信息的手段，而非能力本身缺失。
4. **基准饱和 ≠ 能力饱和。** 感知与知识型基准分数接近天花板时，主动寻找像直觉推理这样未被当前 scaling 覆盖的能力面 — 这正是评测面轮换的实操含义，比在旧榜上内卷更有信息量。
5. **用人类直觉作为免费的天花板校准器。** 93.1% 的人类准确率意味着这类任务天然具备低成本、高信度的 ground truth 校准：凡是"人类一眼可判"的场景，都可以考虑做成评测项或验证器。
6. **视觉系统选型时把"隐含结构推断"列为独立考察项。** 选空间/视频模型（如 [[entities/lingbot-vision-spatial-native-vision-foundation-model-ant]] 这类空间原生路线）时，除了分辨率与感知指标，应单独验证时空/社会结构推断能力 — 这恰是 HSS 探测、而传统基准漏测的维度。

## 关联

- 同属 MLLM 评估体系：与 [[entities/lingbot-vision-spatial-native-vision-foundation-model-ant]]（空间原生视觉基础模型）相邻 — HSS 从评测侧探测直觉视觉推理，LingBot 从模型侧攻空间视觉。
- 与 [[concepts/verifier-paradox]] 所在的 evaluation 方法论族相邻：HSS 是"人类直觉作为天花板"的评测设计样本。
- → [[raw/articles/humanitys-sixth-sense-hss-benchmark|原文存档]]
