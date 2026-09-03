---
title: "Crayotter: Traceable Multi-Agent Workflows for Long-Form Video Editing"
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [crayotter, video-editing, multi-agent, agentic-workflow, long-video, rlvr, grpo, artifact, ustc, multi-modal]
sources: [raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026]
confidence: 0.7
---

# Crayotter: Traceable Multi-Agent Workflows for Long-Form Video Editing

> **Background**：中科大等团队提出的 Crayotter 系统，从系统工程视角将长视频编辑重构为基于工件（Artifact）溯源的多智能体工作流。论文发表于 2026 年，项目已开源。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

## 核心贡献

Crayotter 的核心创新在于将长视频编辑从黑盒生成问题转变为可观测、可定位、可修复的智能体轨迹问题：

- **基于工件的编辑范式**：将 LLM 对话作为唯一状态转变为显式外部工件（检索覆盖率报告、JSON 分析、时间轴规划、过渡规划、工具调用记录、中间渲染输出等），使剪辑动作具备清晰可定位的结构基础。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

- **覆盖率感知的多模态素材检索**：将抽象剪辑请求分解为视觉、叙事、风格等维度的覆盖标签，迭代搜索缺失的语义证据，直到素材池覆盖率达到目标阈值。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

- **基于环境的反射机制**：当工具调用触发诊断失败（时间戳不准确、转场不平滑、旁白未对齐），智能体仅修复受影响的片段而非重启完整剪辑流程。纠错本质不是反复生成，而是局部编辑特定时间轴或调用特定工具。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

- **轨迹级 RLVR 优化框架**：利用 GRPO 算法结合可验证的剪辑信号、LLM 评委评分及人类偏好校准进行优化，表明长视频生成优化需要超越黑盒评分，从底层工具调用准确度、时长匹配度和工件有效性出发重新设计训练目标。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

## 方法论

### 拒绝黑盒：寻找可定位的"工件"

Crayotter 引入带有时间戳水印的技术，将时间坐标直接渲染在感知证据上以绑定语义观察与绝对剪辑坐标。研究阶段的智能体不调用任何处理工具，而是进行深度叙事推理，输出极度详尽的"剪辑蓝图"（包含叙事结构、镜头顺序、节奏、转场和旁白意图）。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

### 纠错本质：基于环境的反射

执行阶段 ReAct Editor 基于蓝图和素材调用超过 20 个模块化视频编辑工具（裁剪、合并、转场、字幕、响度调整等）。错误定位到特定源片段或时间戳跨度，仅修复受影响工件而非重启。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

### 溯源素材：内容覆盖而非盲目生成

素材准备阶段被证明是长视频质量的核心瓶颈——素材缺乏支撑时无论后期工具多强大也无法凭空捏造合理叙事。系统将用户请求扩展为场景、人物/动作、风格等覆盖标签，根据候选视频的边缘覆盖增益重排序，持续搜索直到覆盖率达阈值或预算耗尽。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

## 实验评估

在 23 个固定编辑主题的综合评估中，Crayotter 与 CapCut-Mate 和 CutClaw 基线对比，在主题一致性、内容丰富度、叙事连贯性、剪辑流畅度和视觉质量五个维度上均显著优于基线。^[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026.md]

## 项目资源

- 论文：https://arxiv.org/abs/2606.07636
- 代码：https://github.com/idwts/Crayotter

## 相关实体与概念

- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR (Reinforcement Learning with Verifiable Rewards)]]
- [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]
- [[concepts/multi-agent-collaboration-patterns|多智能体协作模式]]
- [[concepts/agentic-workflow-patterns|智能体工作流模式]]
- [[entities/self-taught-rlvr|Self-Taught RLVR]]
- [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|2026 年强化学习算法综述]]

→ [[raw/articles/crayotter-traceable-multi-agent-long-form-video-editing-ustc-2026|原文存档]]
