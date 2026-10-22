---
title: "手机上的 computer use：Opposable 用 2B 模型在 Android 本机操作 App"
created: 2026-10-22
updated: 2026-10-22
type: entity
tags: [agent, computer-use, on-device, quantization, inference, gui-agent, llama-cpp, vision-encoder]
sources: [raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026]
confidence: 0.8
provenance_state: extracted
---

# 手机上的 computer use：Opposable 用 2B 模型在 Android 本机操作 App

Opposable 是一个 Android 应用：通过 **accessibility service** 读取屏幕、在任意 App 内点击与输入。Claude、ChatGPT 或 Codex 可以从电脑端驱动它；当没有任何外部模型连接时，一个 **2B 参数的屏幕模型在手机本机跑同一套 loop，不经过服务器**。文章（2026-10-21）给出完整的分阶段耗时与优化实测。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md]

## 模型与运行时

模型是阿里巴巴 Mobile-Agent 团队的 **GUI-Owl 1.5 2B**（Qwen3-VL 微调，看图输出单个动作），经 **llama.cpp** 及其多模态库以 **4 bit 量化**运行：语言模型 1.05 GB + 视觉编码器 445 MB。分工上，约四十个常见任务（设闹钟、给联系人发短信、开 Wi-Fi）走**固定代码、不调模型**；此前由大 agent 摸索出的流程则从屏上文本回放。模型只处理 App 无法用其它方式完成的任务。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md]

## 一步的耗时拆解（Mac M2 CPU，4 线程）

一步 = 三阶段：**encode**（截图→图像 token）3.6 s → **prefill**（token + 任务）2.5 s → **decode**（写动作，约 45 token：一句意图 + 一个 JSON tool call）1.1 s，然后点击。未优化时编码器耗时 4.6 s，整步 10.6 s；优化后整步 **7.3 s**，在 36 屏测试集上准确率不变。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md]

三项 CPU 侧优化：^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md]

1. **裁剪状态栏/导航栏/键盘**，在编码前缩小图像。
2. **丢弃平坦背景 token**：编码器之后，不与任何内容相邻的纯背景 patch 被丢弃，其余 token **保留原网格位置**，使模型看到的布局一致；prefill token 减少 **25–40%**，准确率 28/36（不裁剪为 27/36）。实测 400 token 的真实网格为 13×30。
3. **Grounded early exit**：模型先写意图句（如 "Action: Click the Files app icon."），当该句在 accessibility tree 中**恰好匹配唯一元素**时，App 直接点击该元素并停止生成，坐标从不解码——模拟器上这类步骤 decode 从 2–4 s 降到 **0.8–1.3 s**。

## 视觉编码器放到手机 NPU

只把 GUI-Owl 的 **vision tower 导出 ONNX**：24 层 ViT 加 merger，约 0.4B 参数，固定输入 416×960 像素，单屏约 5420 亿次 multiply-accumulate。高通的 AI Hub 为 Galaxy S25（Snapdragon 8 Elite）编译并跑在 NPU 上：**fp16 下 236 ms、873 个算子全部落在 NPU**，约比 M2 的 3.9 s 快 16 倍；8-bit 权重 + 16-bit 激活为 213 ms 更快，但与参考输出漂移（cosine 0.978–0.985 vs fp16 的 0.992–0.996），点击行为随之劣化。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md]

## 可迁移的工程结论

- **端侧 GUI agent 的成本结构里，视觉编码占一半**（M2 上 10.6 s 中 4.6 s 在 seeing）——优化优先级必须从编码器开始，而非解码。
- **屏幕结构化信息（accessibility tree）可以替代坐标生成**：意图句 → 元素唯一匹配 → 提前退出，是最便宜的延迟削减手段。
- **背景 token 剪枝保持网格位置不变**是"省 token 不换布局"的关键实现细节，否则模型的空间对齐被破坏。
- 量化精度选择要在**端侧算子与数值漂移**之间权衡：8-bit 更快但 cosine 从 0.995 掉到 0.98 量级即足以改变动作。

## 关联

- [[entities/anthropic-computer-use-best-practices]] — computer use 的一般工程规范
- [[entities/gemini-35-flash-computer-use-agent-harness]] — 云端模型的 computer-use harness 对照
- [[entities/on-device-harness-qwen38-27b-portable-computer]] — 端侧 harness 的另一条路线
- [[entities/llama-cpp-deployment]] — llama.cpp 多模态端侧部署实践
- [[concepts/inference-optimization]] — 分阶段延迟归因 + 算子卸载 NPU 的优化方法

→ [[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026|原文存档]]
