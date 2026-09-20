---
title: "视觉源幻觉与 ACFT：清华 ACM MM 2026 的 0.9% 数据幻觉修正法"
created: 2026-09-20
updated: 2026-09-20
type: entity
tags: [multimodal, vlm, hallucination, contrastive-learning, fine-tuning, evaluation, tsinghua, acm-mm-2026]
sources: [raw/articles/acft-visual-origin-hallucination-mllm-2026]
confidence: 0.7
review_recommendation: ingest
---

# 视觉源幻觉与 ACFT：清华 ACM MM 2026 的 0.9% 数据幻觉修正法

多模态大模型（MLLM）的物体幻觉长期被归因于语言先验——模型在语料里见惯"餐桌"和"男孩"共现，于是即便图中没有也顺着统计规律说出来。清华大学团队在 ACM MM 2026 论文中提出这一解释并不完整：**当模型输出很短（如仅回答 Yes/No）时，推理更依赖视觉模态，此时会浮现出另一套根植于视觉特征提取本身的幻觉机制**，并将其命名为**视觉源幻觉（visual-origin hallucination）**，对应解法为 ACFT。该方法仅用 COCO 数据集 0.9% 的数据量、不增加任何推理开销，在 POPE、MME 及四个描述级幻觉基准上，于 LLaVA、MiniGPT-4、Qwen2.5-VL 三个模型上取得优异表现。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

## 研究背景：语言先验解释的适用边界

已有工作大多沿"语言先验"线索展开：归因于文本共现统计的过度依赖、定位到专关注文本 token 的"幻觉注意力头"、指向摘要 token 的注意力汇聚、或认为细粒度推理监督不足。缓解方法按干预位置分为几类：VCD、OPERA 等在解码阶段做输入级干预；Woodpecker 借助外部 grounding 模块做输出后处理；RLHF、DPO 一类后训练对齐方法。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

这些方法的共同前提是"幻觉主要来自文本侧偏置"。该假设在长文本输出场景（如"详细描述这张图"）下成立，因为丰富上下文会放大语言偏见；但**当输出退化为"有/没有"的短回答时，语言先验的作用大幅削弱，上述方法效果也随之下降**。短输出场景的幻觉来源因此成为需要单独回答的问题。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

## 第一步：诊断——视觉源幻觉存在吗

研究者在 LLaVA v1.5 上做了两组互补分析并给出量化证据：^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

- **图文嵌入错位**：幻觉样本的图像-文本嵌入余弦相似度显著低于正确样本——正确样本平均 **0.158**，幻觉样本平均 **−0.122**，表明跨模态对齐出现系统性崩塌。
- **注意力模式反转**：用 Smooth Grad-CAM 可视化注意力，并定义"语义合理"分布（目标物体存在时注意力应聚焦目标区域，不存在时应分散）。在 500 个幻觉样本与 500 个非幻觉样本上以归一化香农熵量化后发现系统性违反：**物体存在时熵值高 5.1%**（注意力过度分散而漏掉目标），**物体不存在时熵值低 6.2%**（错误聚焦到无关区域而触发幻觉）。
- **因果验证**：直接干预视觉编码器——施加高斯噪声、降采样、换更弱编码器使 POPE 平均准确率从 0.842 降至 0.739–0.822；换更强的 SigLIP-SO400M 编码器则升至 0.864。这为"视觉特征质量驱动物体存在性幻觉"提供了因果层面支撑。

## 第二步：AHAF——用对抗扰动构造对齐的正负样本

诊断指向视觉错位，最直接的修正思路是对比学习；但普通对比微调（OCFT，匹配图像作正样本 + 随机取一张无关图作负样本）效果不理想，原因是正负样本特征差异不可控、也没有聚焦在目标物体上，模型难以学到究竟是哪些视觉特征触发幻觉。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

为此提出 **AHAF（Adversarial Hallucination Attribute Flipping，对抗幻觉属性翻转）**：用 PGD 在极小的 ℓ∞ 球内对原图施加定向对抗扰动，把一张不引发幻觉的图像"翻转"成引发幻觉的图像。这样构造的正负样本对完全对齐，唯一差异就是那个受控扰动。AHAF 还有副产品价值——它本身是诊断探针：**极小像素级扰动就足以翻转模型答案，说明 MLLM 的视觉表征即便在干净图像上，也已危险地贴近幻觉决策边界**。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

## 第三步：ACFT——对抗对比微调

基于 AHAF 生成的对齐样本对，设计 **ACFT（Adversarial Contrastive Fine-Tuning）**：在嵌入空间中最大化文本锚点与正样本图像的相似度，同时最小化其与负样本图像的相似度。三个工程属性：不依赖特定骨干架构、只需少量数据、完全在训练阶段完成且推理零额外开销。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

## 实验结果

在 LLaVA v1.5-7B、MiniGPT-4 13B、Qwen2.5-VL-7B 三个模型上基于 POPE 与 MME 系统评估（两基准问题全部以 Yes/No 作答，正对应本文关注的短输出场景）。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

| 模型 | POPE 三子集结果 | 相对次优基线 |
|---|---|---|
| LLaVA v1.5-7B | 0.841 / 0.906 / 0.897 | +3.3% / +2.0% / +0.5% |
| MiniGPT-4 13B | — | +3.0% / +5.3% / +2.5% |
| Qwen2.5-VL-7B | 0.864 / 0.875 / 0.884 → **0.877 / 0.900 / 0.916** | 基线本身已很强仍有提升 |

**关键消融（对齐样本对的重要性）**：用同样 3000 张 COCO 图像分别训练 OCFT 与 ACFT，ACFT 在三子集上准确率分别高出 OCFT **35.8% / 7.4% / 17.6%**；尤其在 Adversarial 子集上 OCFT 仅 0.483，**远低于未经微调的原始 LLaVA**——说明不对齐的负样本不仅无益反而有害。相似度差距分析给出解释：ACFT 中目标物体（truck）的正负样本相似度差距明显区别于非目标物体（dog、cat、table），而 OCFT 完全不具这一性质，即 ACFT 让模型学到了"关注目标物体自身特征差异"的一致规则。**可视化**进一步确认两个"病征"被修正：图文嵌入余弦相似度显著提升，Grad-CAM 注意力回归语义合理分布；熵分析显示物体存在案例熵值分别下降 8.7% 和 2.6%，物体不存在案例熵值分别上升 7.4% 和 6.4%。^[raw/articles/acft-visual-origin-hallucination-mllm-2026.md]

## 对 Agent / 模型可靠性工程的启示

- **幻觉归因要分场景**：长输出场景的语言先验解释不能外推到短输出/判官式问答。而 Agent 系统里大量调用正是短判定（工具该不该调、证据是否充分、是否完成），这类判断的失效模式可能与长文描述完全不同。
- **诊断先于修复**：先定位机制（嵌入错位 + 注意力熵反转），再挑干预点（视觉编码器/嵌入空间），而不是直接套解码期干预。这个"诊断→定位→定向修复"的顺序可复用到 Agent 失败分析。
- **对齐的负样本 > 更多负样本**：OCFT 用随机负样本反而低于基线，ACFT 用受控扰动构造的完全对齐样本对才有效——对 Agent 训练/评测数据构造是直接类比：**样本对的唯一变量必须是目标属性**。
- **数据高效 + 零推理开销**：0.9% COCO 数据、仅改训练阶段、推理无额外开销，对已在生产部署的模型是可接受的后训练路径。
- **表征贴近决策边界本身是风险信号**：极小像素扰动即可翻转答案，说明"干净输入上的正确"未必稳健；这类脆弱性探针（AHAF）可直接用于多模态 Agent 的鲁棒性回归测试。

## 相关

- 同类工作（论文覆盖实体）：[[entities/dynhd-diffusion-llm-hallucination-detection-denoising-dynamics-emnlp-2026|DynHD：扩散 LLM 幻觉检测（EMNLP 2026）]]
- VLM 感知与鲁棒性：[[entities/235b参数也没用港中文等发布7模态数据集专测顶级vlm的感知盲区|7 模态数据集测 VLM 感知盲区]]、[[entities/0.25秒识破未见攻击-lod-lvlm-jailbreak-detection-emnlp2026|LOD-LVLM 越狱检测]]、[[entities/covert-vlmaas-covariant-obfuscation-eccv-2026|Covert VLMaaS]]
- 长上下文 VLM：[[entities/llava-onevision-2-full-frame-rate-vlm-glintlab|LLaVA-OneVision-2]]
- 概念层：[[concepts/evaluation-harness-design|评估 harness 设计]]、[[concepts/reinforcement-fine-tuning-rft|RFT]]
- Agent 侧幻觉对照：[[entities/agent-reliability-context-drift-tool-hallucination|Agent 可靠性：context drift 与工具幻觉]]
- 视觉与多模态 MOC：[[moc/vision-multimodal|视觉与多模态 AI]]

## 来源与资源

- 论文：ACM MM 2026（第 34 届 ACM 国际多媒体会议），arXiv `2609.00231`
- 代码：`https://github.com/zxp555/ACFT_MM26`
- 作者：徐沛阳（共同一作，清华大学本科生）、朱小佩（共同一作，清华大学水木学者，合作导师朱军）、朱军教授与胡晓林副教授（通讯作者）
- 原文存档：[[raw/articles/acft-visual-origin-hallucination-mllm-2026|原文存档]]
