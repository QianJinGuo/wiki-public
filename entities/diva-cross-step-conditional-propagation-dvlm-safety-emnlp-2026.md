---
title: "DIVA：面向离散扩散 VLM 的跨步条件传播越狱攻击（清华 × 北邮，EMNLP'26）"
created: 2026-10-19
updated: 2026-10-19
type: entity
tags: [ai-safety, multimodal, diffusion, red-teaming, vlm, jailbreak]
sources: [raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026]
confidence: 0.65
provenance_state: extracted
---

# DIVA：面向离散扩散 VLM 的跨步条件传播越狱攻击（清华 × 北邮，EMNLP'26）

清华大学与北京邮电大学提出 DIVA（Discrete-diffusion Vision-language model Attack）：一种面向离散扩散视觉语言模型（dVLM）的白盒视觉越狱攻击方法。它针对的不是「图片里藏了恶意内容」，而是 dVLM 独特的生成机制——视觉条件在每一轮去噪中都被重新读取，风险语义可以沿生成轨迹反复施加影响。论文把这一现象称为跨步条件传播（Cross-Step Conditional Propagation）。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

## 生成机制的差别决定了攻击面

自回归 VLM 可以粗略理解为「先看图，再从左到右写答案」，视觉信息更接近一次性注入的静态条件；dVLM 则像在一张被遮住的答卷上逐步揭开答案——从大量被遮蔽的 token 状态出发，经多轮去噪恢复完整回复，每一轮都重新使用视觉条件。如果图中存在不易被表面规则发现的风险语义，它就有机会在多个阶段反复影响模型，影响沿着生成轨迹累积。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

## 攻击链路：意图混淆 + 扩散感知优化

DIVA 采用白盒威胁模型：可访问目标 dVLM 的参数、梯度、视觉编码器与训练时使用的遮蔽接口，但**不修改模型权重**，攻击只发生在推理输入端——在原图上叠加受像素幅度约束的扰动，并配合改写后的文本提示，让模型按原本的反向去噪流程生成答案。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

方法由两部分组成。**跨模态意图混淆**把原始风险指令拆到图文两端：文本侧将直接出现的对象、动作、目标替换为依赖图片才能理解的指代表达，被移出的关键语义放进一张复合图像（文字层与相关视觉内容叠加或拼接），单独检查文字或图片都看不到完整意图，只有多模态融合时才重新联系起来。**扩散感知的对抗优化**则从完整去噪轨迹中抽取多个时间步、按各时间步的遮蔽比例构造不同中间回答状态，对每个时间步只计算被遮蔽位置上的预测损失（衡量当前图像能否推动模型恢复预设的非拒答目标），再对各时间步损失取平均，让同一份扰动同时适应多种中间状态——优化的不是某一帧「快照」，而是整段轨迹中的持续影响。梯度依次经过视觉编码器、多模态投影层和双向扩散 Transformer 后回到输入图像，每次更新都把扰动投影回规定像素范围。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

两个设计细节值得注意：优化阶段只用一个很短的肯定式、非拒答前缀作为教师强制目标，目的是推动模型跨过「拒绝还是继续回答」的决策边界，而不是机械复现预写内容，最终效果由模型自由生成结果与独立安全评估器判定；此外 DIVA 不以单条问题为单位学习扰动，而是面对同一风险类别的多条提示联合优化一个**类别级共享扰动**，使其同时适应「多问题」与「多状态」，减少对单一句式的过拟合，推理时只需把离线算好的扰动加到复合图像上（仅涉及图像相加与裁剪），模型结构、权重与标准反向去噪流程均不改变。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

## 实验结果

在 HADES 与 VLSBench 两个多模态安全基准上测试 LLaDA-V、MMaDA、LLaDA2.0-Uni 三类 dVLM，并对比面向自回归模型设计的视觉越狱方法。HADES 的 Beaver 语义评估下平均 ASR：LLaDA-V 58.80%、MMaDA 67.73%、LLaDA2.0-Uni 69.07%，均高于对应模型上的比较方法（MMaDA 上比 DIJA 高 6.67 个百分点，LLaDA2.0-Uni 上高 19.20 个百分点）。在未参与扰动优化的留出分布 VLSBench 上，LLaDA-V 与 MMaDA 的 ASR 仍分别为 52.07% 与 74.16%，同样最优——说明跨步条件传播与生成机制相关，但并非一个扰动可无条件迁移到所有模型（仍须针对目标模型与风险类别重新优化）。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

消融实验显示效果并非「噪声越大越有效」：在 LLaDA-V 上逐步加入复合图像、随机扰动、打乱空间结构的扰动，再到完整 DIVA 优化扰动，平均 ASR 依次为 32.93% → 41.00% → 44.53% → 52.67% → 58.80%；复合图像提供跨模态语义、空间结构贡献一部分效果，真正针对多个去噪状态做优化才带来主要增益。可感知性分析则显示攻击图像与原图在肉眼观察下高度相似，放大差异后才可见低幅度变化。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

## 防御启示与边界

论文把重点放在防御问题上：我们是否只检查了模型的输入与最终答案，却忽略了答案是怎样一步步生成出来的？给出的方向包括：文本过滤与图像过滤协同工作、检查跨模态组合语义；对扩散式模型比较不同去噪区间的安全判断是否异常切换，监测生成过程中的阶段稳定性；为不同生成架构建立专门的红队评测（自回归与扩散的风险接口不同，方法不能照搬）；把安全对齐从最终输出前移到生成轨迹。这些方向可服务于模型上线前的安全验收、红队评测、多模态内容审核与更新后回归测试，对机器人、智能终端等持续处理图像的系统尤为相关。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

边界同样明确：这是白盒研究，论文未公开优化后的扰动文件、可直接复现的攻击脚本、具体步长与扰动预算，因此结论首先说明 dVLM 内部确实存在可被利用的结构性攻击面，不等价于同一扰动可攻击所有商业模型。^[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026.md]

## 关联

- 概念：[[concepts/ai-safety|AI 安全]]
- 相关实体：[[entities/baddlm-diffusion-language-model-backdoor-2026|BadDLM 扩散语言模型后门]]、[[entities/colt-eccv-2026-latent-thought-chain-multimodal-reasoning|多模态潜空间推理]]、[[entities/she-safeevolve-agent-safety-harness-policy-coevolution|SHE 安全策略与 Harness 协同演化]]

→ [[raw/articles/diva-cross-step-conditional-propagation-dvlm-safety-emnlp-2026|原文存档]]
