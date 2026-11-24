---
title: "手机上的 computer use：Opposable 用 2B 模型在 Android 本机操作 App"
created: 2026-10-22
updated: 2026-11-10
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

## 深度分析

### 延迟结构：一半预算花在"看"，优化顺序由成本决定

未优化时一步 10.6 s 的构成是编码 4.6 s、prefill 4.3 s、decode 1.7 s——近半时间花在把截图变成 token。三项 CPU 优化对准的正是这三个阶段：裁剪与背景 token 剪枝压缩编码和 prefill 的输入量，grounded early exit 则干脆跳过 decode（意图句唯一命中 accessibility 元素时坐标从不生成，2–4 s 降到 0.8–1.3 s）。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:56-57,71,87] 值得注意的是 decode 明明最小，却是唯一能被"完全消除"的阶段；而优化后编码仍占 3.6/7.3 s，输入侧压缩有物理下限，这正是把 vision tower 单独搬去 NPU（236 ms，约 16 倍）的动机。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:125]

输入侧的具体账目：裁剪与剪枝把编码输入从 576 个图像 token 压到 400 个，400 token 的真实网格是 13×30，prefill 相应省掉 25–40% token。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:59-66,71,85]

### 结构化信息替代坐标生成

手机场景有电脑端没有的杠杆：accessibility tree。意图句与唯一元素精确匹配即可直接点击目标元素，省掉的不只是时间——被跳过的恰是数值上最易漂移的坐标输出。这与 [[entities/computer-use-45x-more-expensive-than-structured-apis|computer use 比结构化 API 贵 45 倍]] 的结论同向：模型自由生成是最昂贵的通道，能走结构化匹配就不要生成。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:87]

### 分层降级与自检：端侧精度的可靠性边界

NPU → GPU（LiteRT）→ CPU 的启动降级链上，每个候选编码器都要过捆绑截图自检（对 CPU 编码器 cosine ≥0.97 且更快）才被采用，CPU 路径永远兜底。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:145] 精度选择同样以行为一致性划线：fp16（236 ms）是唯一过 0.99 阈值的版本；w8a16 快到 213 ms 但 cosine 落到 0.978–0.985，点击行为随之劣化；全 8-bit/4-bit（0.67–0.90）直接弃用。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:125] 量化在端侧不是压缩率问题而是可靠性问题，与 [[entities/quantization-techniques|量化技术]] 的通用权衡互为实例；640-token 输入编不出 NPU 版本就留在 CPU——能力缺口用回退而非硬切处理。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:145] NPU 编码器返回与 llama.cpp 自己的编码器相同的浮点数组，因此剪枝、网格位置、KV 缓存与提前退出全部原样工作；代价是首次加载要花 72 s 让 NPU 准备计算图（每次安装一次，之后 0.39 s）。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:125,145]

### 子目标分解加逐步核验：执行结构的可靠性红利

8 个 held-out 任务、各跑两轮：整任务一次执行得 4/8 与 3/8；改为一次一个子目标、每个声称的成功先用 accessibility tree 核验再继续，两轮均为 6/8。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:160] 提升不来自模型而来自执行结构，与通用 agent harness 的"计划→验证→推进"闭环一致。同文的 900-token 系统提示词 prefill 一次后复用 KV 缓存，则与 [[entities/llm-prefix-caching-comprehensive-guide|LLM 前缀缓存]] 直接对应。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:71] 分工边界同样是工程决策：约四十个高频常见任务走固定代码不调模型，模型只处理 App 无法用其它方式完成的任务。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:53-54]

## 实践启示

1. **先分段归因再动手优化**：10.6 s 里 seeing 占 4.6 s，不归因就会把力气打在最小的 decode 上。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:57]
2. **端侧 GUI agent 先削编码器**：CPU 侧裁剪+剪枝到 3.6 s 见顶后，下一档在 NPU（236 ms，约 16 倍），而不是继续抠 decode。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:87,125]
3. **有结构化信息就用匹配替代生成**：意图句唯一命中 tree 元素即提前退出，比解码坐标又快又稳。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:87]
4. **端侧精度以任务行为验收，不以压缩率验收**：8-bit 只快一成、cosine 掉出 0.99 即改变动作。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:125]
5. **新硬件路径必须带自检与回退**：NPU/GPU/CPU 三级降级，自检不过就回 CPU，最差路径永远可用。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:145]
6. **"一次一个子目标 + 逐步核验"作为默认执行结构**：held-out 任务从 4/8、3/8 提到两轮 6/8，纯靠结构而非模型。^[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026.md:160]

## 关联

- [[entities/anthropic-computer-use-best-practices]] — computer use 的一般工程规范
- [[entities/gemini-35-flash-computer-use-agent-harness]] — 云端模型的 computer-use harness 对照
- [[entities/on-device-harness-qwen38-27b-portable-computer]] — 端侧 harness 的另一条路线
- [[entities/llama-cpp-deployment]] — llama.cpp 多模态端侧部署实践
- [[concepts/inference-optimization]] — 分阶段延迟归因 + 算子卸载 NPU 的优化方法

→ [[raw/articles/opposable-on-device-2b-computer-use-android-gui-owl-2026|原文存档]]
