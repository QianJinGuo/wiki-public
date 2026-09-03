---
title: "VibeWorlding：多模态Agent自主构建交互式3D世界"
created: "2026-08-31"
updated: 2026-08-31
type: entity
tags: [agent, multimodal, 3d, world-construction, rl, benchmark, open-source]
sources:
  - raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026
confidence: 0.85
provenance_state: extracted
---

# VibeWorlding：多模态 Agent 自主构建交互式 3D 世界

HKUST(广州)+腾讯AI平台部，多模态智能体通过多轮对话+工具调用+渲染反馈自主构建/编辑3D世界。开源6,828 query + 2,616 3D资产 + 323种子世界 + 完整训练管线。^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## 任务形式化

多轮、多模态、工具集成的推理过程：给定多模态请求→智能体推断意图→规划布局→调用3D工具（检索/添加/旋转/平移/删除）→观察沙盒反馈（3D地图文本+五视角渲染图）→循环。^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## VWE-Bench：6,828 条查询

人机协作构建：种子世界生成→扰动生成→查询改写。六大类查询：精确资产级编辑（Verified，可对比Ground-truth）+ 模糊表达/场景批判/引导/重申/复杂描述（Unverified，Rubric打分）。^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## 双重约束验证器

物理可行性（碰撞检测/高度/生态合理性）+ 意图对齐（风格/布局/完整性）。^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## VibeWorlding-Gym 训练管线

1. Gemini 3.1-pro 反向合成冷启动 SFT 轨迹
2. GRPO + 双重约束验证器奖励信号做多模态 RL
3. 纯文本从零构建 + 多模态图文编辑联合学习^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## 关键结果

**VibeWorlder-30B-A3B RL 后训练反超 GPT-5.5 和 Qwen3.8-Max**（综合 Pass@1）。GPT-5.5/Qwen3.8-Max 整体成功率不到 60%。^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

六维能力分析：
- **碰撞无冲突**：所有模型共同瓶颈（RL 后 59%~68%）
- **3D 推理**：RL 提升最显著（基座 6%~20% → RL 后 56%~85%），VibeWorlder 30B 达 85% 超 Gemini 3.1-pro 62%
- **检索能力**：91%~99%，超前沿闭源模型

**结论：RL 让模型看懂场景、找对资产的能力大幅跃升，但精确空间操控仍是所有模型共同天花板。**^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## CLI 原型

命令行交互 + 浏览器实时3D渲染，Agent自我修正（栅栏高度/树冠大小/精确定位删除）。^[raw/articles/vibeworlding-3d-world-construction-multimodal-agent-tencent-2026.md]

## 开源

- 代码：https://github.com/usail-hkust/VibeWorlding-Gym
- 数据：https://huggingface.co/datasets/usail-hkust/VWE-Bench
- 模型：https://huggingface.co/collections/usail-hkust/vibeworlder
