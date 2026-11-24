---
title: "WorldTUI：360M 模型把终端学成世界模型——屏幕网格 token、333 词表与 DAgger 式自我纠错"
created: 2026-10-24
updated: 2026-11-10
type: entity
tags: [world-model, small-model, fine-tuning, terminal, agent-ui, dagger, decoding-constraints, on-device, webgpu]
sources: [raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026]
confidence: 0.75
provenance_state: extracted
---

# WorldTUI：360M 模型把终端学成世界模型——屏幕网格 token、333 词表与 DAgger 式自我纠错

> SimEdw 的个人项目（2026-10-21）：把 Qwen2.5-Coder-0.5B 微调成一个"终端世界模型"，背后**没有 shell、没有文件系统**——只有按键输入和模型对屏幕变化的预测。它能建文件、跑 vim/tmux、玩 snake，208 MB 模型在浏览器 WebGPU 里跑。作者自述两个最大收益点是"教会模型修自己的错"和"并行解码屏幕行"。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

## 为什么把终端当作世界模型来做

作者原本想要"应用程序不是写出来的而是学出来的"——不由游戏引擎渲染画面，而由模型根据用户输入预测下一帧。没有 GPU 集群和视频数据，就把野心缩到终端：终端是**字符网格而非像素**，可以靠运行真实 shell 无限生成完美标注的训练数据，而预训练语言模型本身已经"见过"大量 CLI 用法。真正要验证的问题是：模型是维持了一个**一致的世界**，还是只生成了看起来像终端的输出。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

这个问题的提法很关键：GUI/终端 agent 的评测一直依赖真实环境回放，而"学出来的环境"把评测从"能不能操作"变成"能不能预测"——后者可离线、可批量、可解释。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

## 表示：屏幕即状态，动作用"行重写"而非"续写 token"

标准 LLM 是"给 prompt 然后往末尾追加 token"，这与终端不匹配：一次按键可能只在光标处打一个字，也可能清屏并把光标送回顶部（vim 里尤其常见）。所以作者不做文本续写，而是预测**屏幕变化**：把终端看作字符网格，每个 cell 一个字符；模型看到的不是网格本身，而是一串描述"屏幕内容 + 这次改了什么"的 token。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

每个会话以快照开始：屏幕大小、内容、光标位置。尺寸是输入的一部分而不是固定的 32×96——demo 里可以自定义行列数，也可以从 bash prompt、vim 缓冲区或一局 snake 的中途接续（作者注：从 bash 起步时效果最好）。之后每次按键以**真实终端会发出的原始字节**输入，模型回答"要重写哪几行、光标最后在哪"。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

关键工程技巧是**不要重发整屏**：滚屏时若让模型重写所有行，一次按键要花约 3,000 token；改成"行操作"（copy 行 / 覆盖部分行 / 插删行 / 清屏 / 翻 man page）后，滚屏每行只需 3 token 而不是 ~96。最终词表只有 **333 个 token**：256 个 byte token + 64 个 row token + 少量特殊操作；坐标这种小数字也能用单 token 表示（如 17 只占 1 token）。解码用**语法约束**，保证每次预测都是合法的屏幕更新。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

时间也是输入：没有按键时给模型一个带时长的 WAIT token，这也是 snake 能自己动起来的原因。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

## 数据：自造 33.6 万段会话，约 20 亿 token

现成数据集（如 asciinema）缺少"按键→屏幕"的精确对齐，所以作者自己在 Docker 里跑真命令生成会话：随机化用户名/主机名/prompt 风格/终端尺寸（主要 32×96，也有 24×80、40×120）；按虚构人设（developer、sysadmin）随机化目录与文件；挑 6–24 个加权活动（改文件、vim、python、tmux 等）；维护状态让会话"打开的文件大多真实存在"；按人类节奏逐键输入并带 tab 补全，**故意设 1.2% 的 typo 率**；**35% 的会话从中间开始**，迫使模型学会从任意状态接手。最终 **336k 段会话 ≈ 2B token**——对比基座模型训练时见过的约 18T token。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

## 训练：先 SFT，再用 DAgger 学"自我修复"

第一轮是朴素 SFT：给屏幕历史和按键，预测真实终端产生的屏幕增量。普通打字学得很快（字母出现、退格删除、光标移动），但长会话里**误差会累积**：文件名打错一次，错误就顺着后面整个会话传播。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

作者借了 DAgger 的思路——从模型自己会走到的状态学，而不只学理想演示：在录制会话里挑一个点，让模型接管、走偏，再构造一个"下一次按键就把屏幕修回来"的修正样本，用修正后的帧加之后几个真实事件继续训练。这里有一条硬约束：**修正引入的词必须在此前真实屏幕或用户输入里出现过**，不能来自模型自己的预测——否则等于奖励幻觉。约**三分之一**的采样会话能产出有用的修正样本，作者按此迭代了三轮，每轮都从最新模型的错误里取数据。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

效果：自我修复是收益最大的训练改动之一，配合额外数据把自建基准 **PresetBench 从 24/47 提到 38/47（int4 量化下）**。整体训练与后续精炼合计约 **75 GPU 小时（单张 RTX 4090）**。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

## 工程含义

1. **窄词表 + 结构化动作是"小模型做环境模型"的核心杠杆**：把"重写整屏"换成"行操作"、把状态变化压成 333 词表并加语法约束，等于把输出空间从"任意文本"缩到"合法屏幕更新"，这是 360M 能在浏览器里跑的前提。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]
2. **误差累积是长程 agent 环境的头号问题，DAgger 式修正数据是可复用的解法**：环境模型和真实 agent 一样会"错一步、错到底"；把"修回来"当成显式训练目标（且限定在可解释信息范围内）比单纯加大数据更有效。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]
3. **合成数据的可信度来自"可验证性"**：Docker 里跑真命令得到的标签天然正确，随机化人设与活动分布保证覆盖，1.2% typo 与中途接续是刻意制造的难度。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]
4. **局限**：这是"终端世界"的模拟器，不是真实执行环境——它能预测屏幕，但不保证副作用正确；把它当作 agent 评测/训练环境时，需要与真实 shell 的抽样对齐（作者未给出与真实终端的分布偏移量化）。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md]

## 深度分析

### 一致性来自表示而非规模：333 词表把输出压成"合法屏幕更新"

最容易被低估的决策是把世界状态做成 64 行格子 + 行级操作：滚屏从约 96 token/行压到 3 token/行，词表收缩到 256 byte + 64 row 加少量特殊操作。360M 参数因此够用——模型要学的不是"什么是终端"，而是格子间的转移函数；表示与动作设计是比参数量更硬的杠杆。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:76-80]

### 行并行的边界：先拿一致性换速度，再用自投机赎回

行并行把解码成本从"所有行之和（至多 3,000 步）"降到"最长一行（约 100 步）"，大帧 p95 从 6.7s 降到 0.87s；但训练要配套行间注意力掩码，是架构级改动而非纯推理技巧。行独立也带来真实失败：内容开在错误的行、snake 被多行同时加食物；解法是同一模型自草稿自校验的自投机解码，浏览器里大更新从 2.5s 回到 0.9s。每一步性能优化都要回答"它破坏了哪条约束"。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:180-186]^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:192-208]

### 蒸馏揭示的记忆定位：通用准确率掩盖世界状态丢失

嵌入表占基座 151,936 词表的 136M 参数，砍到 333 token 后模型从 494M 缩到 360M（990→716MB，int4 后 208MB）。蒸馏给出全文最锐利的发现：剪掉第 7 层，next-token 准确率仅从 99.3% 降到 98.9%，精确读回却从 64/64 崩到 0/64——"还会打字"与"还记得世界"是两种能力。用第 23 层顶替第 22 层的学生读回 31/64，恰为随机水平，480M token 也未恢复；保留 1/7/15/22 层的 16 层学生读回 64/64，ShellBench 却从 78 掉到 65。作者最终放弃蒸馏：长程记忆嵌在具体层里，删层会悄悄破坏它，通用指标测不出。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:212-216]^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:220-220]^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:222-226]

### 对世界模型路线的启示

1. **一致性要有专门探针**：跨会话读回测试（写两个值、分别读回）比 next-token 准确率更能暴露"假世界模型"；这与 [[entities/quantization-techniques|量化]] 里"困惑度不降但能力已损"的教训同构。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:218-224]
2. **性能与一致性是两张支票**：行并行赢回延迟、输掉行间协调，自投机再把一致性赎回；[[concepts/speculative-decoding|投机解码]] 在此是同模型自校验而非大小模型协作。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:180-208]
3. **作者自认这是"相当弱的世界模型"**：它维持着一个大体一致的世界，却没有实现任何底层规则——这为"学出来的环境"的能力边界提供了诚实锚点。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:280-284]
4. **下一步实验最有信息量**：在数千个随机生成、各有控件与规则的应用上训练，检验模型能否凭上下文线索预测从未见过的应用——这是"世界模型能否零样本泛化到新环境"的直接实验设计。^[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026.md:282-284]

## 关联

- 世界模型主线：[[concepts/world-models]]、[[entities/code-world-model-coding-agent-as-world-brain-2026]]、[[entities/baai-orca-next-state-prediction-world-model]]
- 终端/GUI agent 环境：[[concepts/agent-harness-engineering-paradigm]]

→ [[raw/articles/worldtui-terminal-world-model-360m-simulated-shell-2026|原文存档]]
