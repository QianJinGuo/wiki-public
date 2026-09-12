---
title: "ControlFoley：多模态可控视频音效生成（ACM MM 2026，小米）"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [multimodal, video-generation, audio-generation, foley, controllable-generation, diffusion, xiaomi, acm-mm-2026, comfyui, open-source]
sources: [raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026]
confidence: 0.8
provenance_state: extracted
---

# ControlFoley：多模态可控视频音效生成（ACM MM 2026，小米）

ControlFoley 是小米集团（共同第一作者杨剑轩、郭新月，通讯作者杨剑轩，武汉大学合作）提出并已被 ACM MM 2026 接收的多模态可控视频音效（Foley）生成模型，论文、代码与模型权重均已开放。它要解决的核心问题不是「视频能不能自动配出声音」，而是「用户不想要这个声音时，能不能告诉模型换一种」——把视频动作、文本提示与参考音频都变成可用的控制条件，让生成音效既跟得上画面，又贴近创作者想要的语义与风格。^[raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026.md]

## 控制条件的角色分工

ControlFoley 的关键设计理念是让不同条件「各司其职」，而不是简单拼接：视频主要负责约束动作节奏与发声时刻，文本负责补充或改写声音语义，参考音频负责提供音色与质感，同时抑制其时间结构被一并复制。围绕这一点，模型采用三项关键设计：联合视觉编码（CLIP 对齐语义 + 自研时空音视频编码器 CAV-MAE-ST 强调动作变化与音画时序）以增强视觉语义与时序表征，并缓解文本控制中的视文语义冲突（抑制视觉条件对文本指令的过度主导）；参考音频的时间-音色解耦，保留音色与质感、抑制不应迁移的节奏结构；多模态 Transformer 统一生成与鲁棒训练（随机模态丢弃 + REPA 表征对齐），使同一套模型兼容基础视频配音、文本辅助、文本控制、参考音频控制与纯文本音效生成。^[raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026.md]

## 评测：可控性与同步性的取舍

ControlFoley 的评测围绕「声音是否符合语义、是否贴合画面、是否与动作同步、音频质量是否稳定」展开，指标包括 CLAP（音频-文本语义匹配）、DeSync（音画时间偏移，越低越好）、IS（音频质量），参考音频控制任务额外比较音色相似度。在基础视频配音评测中，ControlFoley 在 VGGSound-Test、Kling-Audio-Eval 与 MovieGen-Audio-Bench 三个测试集上相对最佳开源基线同时提升语义对齐、音画同步与音频质量；在可控任务上，当文本与视频冲突增强时仍能更稳定地跟随文本意图，使用参考音频时能迁移声音风格而不照搬其时间节奏。^[raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026.md]

## 生态与部署入口

项目的工程化程度较高，覆盖从快速体验、可视化创作到研究复现与本地部署的不同需求：零部署在线体验（项目主页 Demo / Hugging Face Space）、创作者工作流（官方 ComfyUI 节点与全任务工作流）、智能体快捷调用（通过 Skill 接入 Agent 工作流）、开发者本地部署（GitHub + Hugging Face 官方 Python 实现，或社区 audio.cpp 的原生 C++ 推理与 GGUF 模型）。技术报告见 arXiv:2604.15086，代码见 github.com/xiaomi-research/controlfoley。^[raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026.md]

## 意义

ControlFoley 代表了多模态音视频生成中「从自动补全到可控生成」的一步：把控制条件显式建模为语义/时序/音色三个正交角色，并为参考音频做时间-音色解耦，避免「控制即复制」的常见失败模式。它与音频基础模型 [[entities/cvpr-2026-highlight-清华打破多模态音频生成的通才困境omni2sound-音频基础模型开源|Omni2Sound]] 的「通才化」路线互补，也与 [[entities/ard-agentic-autoregressive-diffusion-for-long-video-consistency|ARD 长视频一致性]] 这类「先保证时序一致、再谈控制」的视频生成工作形成对照：前者解决音频侧可控，后者解决视频侧一致。对实际生产链路（AI 视频、漫剧、游戏与广告）而言，可直接落地的 ComfyUI 工作流与 GGUF 部署路径降低了接入成本。^[raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026.md]

→ [[raw/articles/controlfoley-controllable-video-sound-generation-xiaomi-acm-mm-2026|原文存档]]
