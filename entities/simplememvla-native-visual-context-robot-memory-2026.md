---
title: "SimpleMemVLA：原生视觉上下文即记忆，流式推理把机器人决策延迟压到 0.68 秒"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [vla, robot-memory, multimodal, streaming-inference, kv-cache, embodied-ai, context-learning, openbmb, minicpm, robot-manipulation]
sources: [raw/articles/simplememvla-native-visual-context-vla-memory-2026]
confidence: 0.75
provenance_state: extracted
---

# SimpleMemVLA：原生视觉上下文即记忆，流式推理把机器人决策延迟压到 0.68 秒

SimpleMemVLA 由面壁智能联合华中科技大学、中国人民大学、清华大学、北京大学、北京智源人工智能研究院和北京中关村学院共同提出，核心主张是**不增加专用记忆模块**，直接把带时间戳的视觉历史当作上下文交给 VLA 主干模型，让机器人「记得住过去」而不必先把历史压缩成另一套记忆表示。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

支撑这条路线可行性的关键账本是上下文预算：按论文设置，60 秒历史仅占约 5.6k token，而主干模型支持 262k token 上下文，分钟级视觉历史可以整体放入模型。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

## 核心命题：「写入时承诺」的代价

论文把专用记忆机制的结构性缺陷命名为**「写入时承诺」（write-time commitment）**——在把信息存入记忆的那一刻，系统就必须决定保留什么；而一个细节是否重要，往往要到后续任务中才知道。提前筛选、压缩或覆盖历史，可能丢掉未来才需要的信息。SimpleMemVLA 的替代方案是保留原始历史画面、时间顺序与明文时间戳，让模型在理解当前任务时**一并理解过去**，而不是先加工成固定摘要再据此行动。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

## 架构与信息通路

历史画面沿用视觉语言模型预训练时的视频格式，按时间顺序组织并附上明文时间戳；当前腕部画面单独一路输入。模型据此生成一句当前子任务（例如「揭开红色积木上的盖子」），而**生成这句话时形成的上下文隐藏状态**，会与词元嵌入、机器人本体状态一起传给动作头，生成下一段动作。消融实验表明，历史信息主要通过这份隐藏状态影响行为，而不只是通过子任务的字面文字。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

## 实验结果

**记忆基准。** 在 RoboMemArena 的 26 项任务（轨迹平均超过 1000 个控制步）中，使用 126 秒历史窗口时 SimpleMemVLA 全阶段任务成功率达到 **63.6%，比此前最强模型提高 17.4 个百分点**；其中计数任务从 31.4% 提升至 71.4%，遮挡任务从 39.1% 提升至 64.3%。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

**记忆机制隔离实验。** 研究者固定训练数据、主干模型、子任务监督、动作头和优化器，**只替换记忆机制**（RoboMME），用以区分主干能力与记忆机制的贡献——结果支持「让模型结合当前任务理解仍然可读的历史，比提前决定哪些信息值得留下更有效」。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

**消融。** 在 RMBench 盖积木任务中，30 秒窗口仍包含约 25 秒前的遮盖过程，模型能完成任务；窗口缩至 15 秒、关键画面移出后任务失败。按键计数任务随历史窗口缩短逐步退化；只保留当前画面或打乱帧顺序时，两项任务均失败——关键画面与时间顺序都不可缺少。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

**视觉上下文学习。** 将另一条执行轨迹的片段接入原有历史（换掉积木被盖住的位置、把颜色提示从品红换成红色、替换演示序列的开头动作），模型会相应改变行为，且**推理时没有参数更新**。遮蔽关键历史会影响决策（选错盖子、少算一次并追加执行），对等长度的无关片段进行遮蔽时输出保持不变。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

## 流式推理：长历史不等于长等待

长历史不必意味着长时间等待。通过**复用历史计算**——连续两次决策共享大部分历史，机器人在执行当前动作时提前计算下一次仍会使用的历史前缀并保存键值缓存（KV cache），下一次决策只需处理新增内容并生成子任务——SimpleMemVLA 把 60 秒历史的决策延迟从 **1.02 秒降至 0.68 秒**，低于实测任务的 0.96 秒实时预算（单张 H100、bf16、batch size 为 1）。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

在更长上下文的延迟测试中，约 45 分钟历史对应 245k token：全量重算需 32.1 秒，而流式决策路径缩短至 **1.18 秒**；超长上下文下后台预填充与缓存开销仍会随历史增长。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

## 真机与展望

在真实双臂机器人上，机器人先用相同的黑盖遮住三块彩色积木，再按指定颜色顺序揭盖——此时当前画面已经看不到积木颜色，必须依赖遮挡前的视觉历史判断每个盖子下是什么。三次不同布局的自主执行中揭盖路径随颜色位置改变，第三段采用「中→远→近」顺序，说明模型在利用历史中的颜色与位置关系而非固定扫描方向。团队后续计划引入人类第一视角（ego）数据、机器人示教视频与自主执行历史，让上下文不只帮机器人记住过去，也成为学习和适应的来源。^[raw/articles/simplememvla-native-visual-context-vla-memory-2026.md]

## 资源

- 论文：<https://arxiv.org/abs/2609.05533>
- 开源代码：<https://github.com/OpenBMB/SimpleMemVLA>
- 开源模型：<https://huggingface.co/collections/yinchenghust/simplememvla>
- SimpleMemVLA 同时是 **MiniCPM-RobotManip** 中上下文记忆与流式记忆的支撑技术。

## 相关

- [[entities/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026|LIOS：云机器人基础设施与 VLA sim2real]]
- [[entities/lingbot-vla-2-60000h-open-source-vla|LingBot-VLA 2：6 万小时开源 VLA]]
- [[entities/lerobot-v060-imagine-evaluate-improve|LeRobot v0.6.0]]
- [[entities/egosuite-open100k-embodied-human-video-data-open-source-2026|EgoSuite-Open100K：人类行为数据与具身 Scaling]]
- [[concepts/agent-memory-system-design|Agent 记忆系统设计]]
- [[concepts/context-management-agent-systems|Agent 系统的上下文管理]]
- [[entities/agent-memory-four-schools-comparison-2026-07-22|Agent 记忆四流派对比]]
- [[entities/context-engineering-three-memory-paradigms-comparison|三种记忆范式对比]]
- [[moc/memory-context-systems|MOC：记忆与上下文系统]]

→ [[raw/articles/simplememvla-native-visual-context-vla-memory-2026|原文存档]]
