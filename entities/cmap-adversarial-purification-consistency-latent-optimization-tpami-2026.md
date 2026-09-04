---
title: "CMAP：基于一致性模型的对抗净化（隐空间流形优化，TPAMI'26）"
created: 2026-09-04
updated: 2026-09-04
type: entity
tags: [adversarial-robustness, adversarial-purification, diffusion, consistency-model, latent-space, manifold, image-classification, tpami, defensive-machine-learning, security]
sources: [raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026]
confidence: 0.72
provenance_state: extracted
---

# CMAP：基于一致性模型的对抗净化（隐空间流形优化，TPAMI'26）

## 核心洞察

CMAP（Consistency-Model based Adversarial Purification）把对抗净化从传统的"输入空间加噪—去噪"重新定义为**隐空间流形恢复**问题。关键观察是：生成模型产生的样本与干净数据分布距离较近，而对抗样本存在更明显的分布偏移——因此预训练生成模型的隐空间可作为恢复可信样本的先验。^[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026.md]

该方法发表于 TPAMI 2026《Adversarial Purification by Consistency-aware Latent Space Optimization on Data Manifolds》（arXiv 2412.08394），核心是在分类器架构或攻击类型未知的条件下仍能高效防御。^[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026.md]

## 为什么用一致性模型而非扩散模型

选择一致性模型作为生成先验的原因有二：其一，相较需多步采样的扩散模型，一致性模型能以极少步甚至单步完成生成，降低隐空间优化开销；其二，其基于 ODE 的确定性映射减少了生成随机性对梯度优化的干扰。^[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026.md]

这与传统扩散式净化（先加噪声再反向扩散恢复）形成对比——高斯噪声与对抗扰动分布/作用机制不一致，固定噪声尺度难以适配不同攻击强度，从而加剧鲁棒性与语义保持的权衡。CMAP 转向隐空间优化则绕开了这一矛盾。^[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026.md]

## 三模块检测框架

1. **感知一致性恢复模块**——从隐分布采样 K 个隐向量，联合 MAE 与 SSIM（像素级+结构级信息）构造感知一致性恢复损失，寻找与输入视觉内容一致的隐空间特征。
2. **隐分布一致性约束模块**——约束多个优化隐向量的统计分布，使其与原始隐分布一致；这一守卫生效的前提是 Theorem 1：即便输入空间扰动很小，沿 ODE 轨迹映射到隐空间后也可能引起显著均值偏移。
3. **隐向量一致性预测模块**——将 K 个生成样本分别输入分类器后投票，获得最终预测。

每个模块承担一个职责：感知一致性负责保留语义，隐分布一致性限制结果进入异常区域，投票聚合抵消不同初始化的局部恢复差异。^[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026.md]

## 理论保障

- **Theorem 1**：限制隐空间分布对避免重新拟合对抗扰动是必要的（ODE 映射会放大均值偏移）。
- **Proposition 1**：CMAP 联合优化目标能收紧生成结果相对于原始干净样本的重建误差上界。

两者共同推动样本回归可信数据流形。^[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026.md]

## 实验表现

在 CIFAR-10 与 ImageNet-100 上覆盖 PGD、AutoAttack、BPDA、自适应攻击等威胁模型：

- CIFAR-10 PGD+EOT：相对最强基线鲁棒准确率最高提升 **18.73%**
- CIFAR-10 AutoAttack：最高提升 **6.54%**
- ImageNet-100 PGD+EOT：提升 **6.47%–8.79%**

## 关联

- [[entities/adversarial-verification]] — 对抗验证（属性/形式化层面），与 CMAP 的"净化"形成防御侧互补
- [[concepts/ai-security-landscape]] — AI 安全全景，CMAP 属对抗鲁棒性/防御子域
- [[entities/ai-agents-security-survey-attack-defense]] — Agent 攻击与防御综述，含对抗样本主题
- [[concepts/agent-security-threat-models]] — 威胁模型分析框架

→ [[raw/articles/adversarial-purification-cmap-consistency-latent-optimization-tpami-2026|原文存档]]