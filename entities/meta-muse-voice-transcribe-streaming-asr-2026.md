---

title: "Meta Muse Voice Transcribe 实时语音感知模型"
created: 2026-09-03
updated: 2026-09-05
type: entity
tags: [meta, muse, asr, speech-recognition, streaming, multimodal, real-time, diarization, reinforcement-learning]
sources: [raw/articles/meta-muse-voice-transcribe-streaming-asr-2026]
confidence: 0.8
---

# Meta Muse Voice Transcribe 实时语音感知模型

## 核心信息

Muse Voice Transcribe 是 Meta Superintelligence Labs 开发的首个实时音频感知模型，于 2026 年 9 月 1 日发布。在 Artificial Analysis 流式语音转文本和公开说话人分离基准测试中排名第一。 ^[raw/articles/meta-muse-voice-transcribe-streaming-asr-2026.md]

**核心能力：**
- 实时流式 ASR（语音识别）
- 20+ 说话人分离（diarization）
- 端点检测（endpointing）
- 多语言支持（70+ 语言训练，25 语言验证）
- 无缝语码切换（code-switching）
- 语言/关键词/上下文偏置（context biasing） ^[raw/articles/meta-muse-voice-transcribe-streaming-asr-2026.md]

## 架构设计

Muse Voice Transcribe 是 Muse Spark 家族的自回归多模态模型。

### 流式 ASR

音频以 80ms 块（12.5 Hz）处理，每个块转换为单个 soft token。每个音频块，模型决定继续监听下一音频块或输出文本 token。 ^[raw/articles/meta-muse-voice-transcribe-streaming-asr-2026.md]

- `<|next_audio|>`: 继续监听
- `<|empty_audio|>`: 音频流结束
- **自适应延迟（Adaptive Delay）**：通过强化学习动态调整每个词的延迟。WER 奖励和延迟奖励相乘组合。实现速度-精度 Pareto 最优。

### 说话人分离

- `<|start_of_turn|>`: 标记说话人切换
- `<|speaker_{A-Z}|>`: 区分说话人
- 与 ASR 联合训练，额外奖励信号

### 端点检测

- `<|speech_onset|>`: 语音开始
- `<|speech_endpoint|>`: 语音结束
- 与 ASR 联合训练

## 性能指标

- **流式 ASR WER**: 3.1%（Artificial Analysis 排名第一）
- **说话人分离错误率**: 17.5%（AMI-IHM + AMI-SDM + VoxConverse 平均）
- **Pareto 最优**: 在时间-最终转录精度权衡中达到 Pareto 前沿

## 技术创新点

1. **自适应延迟**：RL 训练的延迟-精度权衡，每个词动态调整
2. **统一 token 架构**：ASR + diarization + endpointing 共享同一 token 空间
3. **70+ 语言 + 无缝 code-switching**：支持中英混合等多语言场景
4. **80ms 音频块处理**：12.5 Hz 采样率的实时处理 ^[raw/articles/meta-muse-voice-transcribe-streaming-asr-2026.md]

## 与 Muse Spark 系列的关系

Muse Voice Transcribe 属于 Muse Spark 家族，与 [[entities/meta-muse-spark-voice-mode-meta-glasses|Meta Muse Spark 语音模式]] 共享底层架构。语音模式专注 Meta 眼镜上的多模态交互，而 Voice Transcribe 是通用实时 ASR 系统，可用于更广泛的音频感知场景。
→ [[raw/articles/meta-muse-voice-transcribe-streaming-asr-2026|原文存档]] ^[raw/articles/meta-muse-voice-transcribe-streaming-asr-2026.md]