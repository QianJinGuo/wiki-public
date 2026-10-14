---

title: "TRACE：面向实时人机交互的流式情绪理解框架（蚂蚁×浙大）"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [ai, multimodal, emotion, streaming, llm]
sources: [raw/articles/trace-streaming-emotion-understanding-emnlp26-2026-10-08]
confidence: 0.75
provenance_state: extracted
---

# TRACE：面向实时人机交互的流式情绪理解框架（蚂蚁×浙大）

## 问题设定

蚂蚁集团与浙江大学的研究团队（EMNLP 2026 Main）指出：现有情绪理解研究大多采用**离线设置**——模型先获得完整视频/音频/对话文本再输出标签。但实时视频对话、具身交互和远程会议要求系统在**信息尚不完整时形成判断**，并随新证据及时调整 → [[raw/articles/trace-streaming-emotion-understanding-emnlp26-2026-10-08|原文存档]]。 ^[raw/articles/trace-streaming-emotion-understanding-emnlp26-2026-10-08.md]

三个关键挑战：信息未完整却需及时判断（语调/停顿/语速在句结束前已传递情绪线索）；模型高置信不等于判断可靠；频繁调用多模态推理代价高。 ^[raw/articles/trace-streaming-emotion-understanding-emnlp26-2026-10-08.md]

## 方法与数据

- **TRACE 框架**：持续跟踪模型情绪判断的变化过程（轨迹可靠性门控），让情绪理解兼顾识别效果与响应效率
- **StreamMER 数据集**：自建的流式情绪理解评测集
- 验证于 StreamMER、MELD、MER2024 三个基准

## 结果

在 StreamMER 上，TRACE 将仅使用在线音频判断的准确率从 **59.87% 提升至 69.47%**，同时显著减少了多模态推理调用次数——流式设定下的「判断轨迹跟踪」兼顾了准确率与推理成本。 ^[raw/articles/trace-streaming-emotion-understanding-emnlp26-2026-10-08.md]

## 关联

- → [[concepts/inference-optimization|推理优化]]（减少多模态推理调用的流式效率路线）
- → [[concepts/attention-mechanism|注意力机制]]（时序信息累积下的证据加权）
