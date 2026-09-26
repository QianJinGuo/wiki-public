---
title: "Story2Video: 从故事文本到可发布短视频的 AIGC Pipeline"
created: 2026-09-01
updated: 2026-09-26
type: entity
tags: [aigc, video-generation, pipeline, ai]
sources: [raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Story2Video: 从故事文本到可发布短视频的 AIGC Pipeline

Story2Video 是 AWS 团队推出的一个端到端 AIGC pipeline，将故事背景、故事线、风格参考图、人物参考图和角色音色自动转成可发布的短视频。它不是简单的文生视频 demo，而是一个小型内容生产系统，需要同时处理脚本、分镜、角色一致性、TTS、口型同步、字幕、转场和音画对齐。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

## 核心问题

系统主要解决三个结构性问题：^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

1. **结构不稳定** — LLM 输出的自然语言难以被下游模块消费。Story2Video 把 LLM 输出约束为结构化 `StoryScript` 和 `StoryShot`，每个镜头包含 `audio_type`、`speaker`、`dialogue_text`、`narration`、`visual_prompt`、`camera_motion`、`character_id` 和 `duration` 等明确字段。
2. **角色一致性** — 多镜头视频中角色外貌容易漂移。通过人物参考图 + Qwen Image Edit 走图生图链路，纯场景镜头走 Z-image 文生图，在灵活生成的同时保留角色一致性。
3. **音画同步** — 对话镜头需要 TTS → lip-sync；旁白镜头需将音频铺到时间轴；最终统一生成 SRT、合并音轨、裁剪时长，生成 `final_video.mp4`。

## 五层架构

| 层 | 组件 | 职责 |
|---|---|---|
| 交互层 | `gui/vibe_video_ui.py` (Gradio) | 用户上传素材、填写故事背景和故事线，生成 session，后台线程不阻塞 |
| 编排层 | `story_to_video_pipeline.py` | 核心入口，串联图像分析→脚本→分镜→TTS→视频→合成，返回 `StoryPipelineResult` |
| 数据协议层 | `story_shot.py` (`StoryShot`/`StoryScript`) | 把故事创意转成机器可执行的中间表示，后续模块读字段而非解析自然语言 |
| 生成能力层 | `story_writer.py` / `comfyui_client.py` / `tts_fish_speech.py` | Bedrock 多模态脚本生成、ComfyUI 图/视频 workflow、Fish-Speech TTS |
| 后期合成层 | `video_editing.py` | 拼接视频、替换音轨、烧录字幕、处理转场和时长漂移 |

^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

## Pipeline 阶段

**Stage 0 — 输入图像分析**：调用 `analyze_all_images`，对风格参考图提取视觉风格/情绪/色彩/光线，对人物参考图提取外貌/服装/表情/姿态。分析结果作为 prompt context 注入脚本生成。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

**Stage 1 — 故事脚本生成**：`generate_story_script` 组合背景、故事线、参考图和分析结果，要求 Bedrock 返回严格 JSON。自动识别用户文本中的时长要求反推分镜数量；自动替换高风险运镜（`pan_left`/`pan_right` → `static`/`zoom_in`/`zoom_out`）。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

**Stage 2 — 分镜图生成**：有 `character_id` + 人物参考图的镜头走 Qwen Image Edit（图生图），无角色镜头走 Z-image（文生图）。双人镜头传入第二张人物参考图。线程池并发，默认并发数 2。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

**Stage 3 — TTS 语音生成**：`audio_type` 路由：`dialogue` 用角色音色（从 `character_voice_map` 查找），`narration` 用旁白参考音频，`bgm_only` 不生成 TTS。音频保存为 `audio/shot_XXX_tts.wav`。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

**Stage 4 — 分镜视频生成**：`dialogue` 镜头调用 MultiTalk（lip-sync），其他镜头调用 Wan2.2 image-to-video。通过 `LegacyShot` 兼容旧接口。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

**Stage 5 — 最终合成**：生成 `subtitles.srt`，收集所有分镜视频，合并 TTS 音频（短于镜头补静音，长于则截断），按顺序拼接、替换音轨、烧录字幕、修正时长漂移。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

## 关键技术栈

- **LLM**：Amazon Bedrock（Claude Sonnet 4.5）用于图像理解和脚本生成
- **图像生成**：ComfyUI（Z-image 文生图 + Qwen Image Edit 图生图）
- **视频生成**：Wan2.2 image-to-video + MultiTalk lip-sync
- **TTS**：Fish-Speech 本地 API 模式，支持角色音色和旁白音色
- **合成**：FFmpeg + MoviePy + pydub

## 部署要点

生产建议将 Gradio UI、ComfyUI、Fish-Speech 拆成三个独立服务。ComfyUI 运行在 GPU 机器上，Fish-Speech 可独立部署在另一张 GPU 或同机不同端口。输出目录挂载到持久化磁盘，多人场景加鉴权，耗时任务可替换为 Celery/RQ 队列。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

## 深度分析

### 结构化中间表示是整条 pipeline 的"承重墙"

从工程视角看，Story2Video 最值得复用的设计不是某个生成模型，而是 `StoryShot`/`StoryScript` 这一层薄薄的数据协议。原文反复强调：prompt 明确要求 `dialogue`、`narration`、`bgm_only` 三者互斥，这让 TTS、图像、视频、合成四个模块都能直接按字段分支，而不是各自再解析一遍自然语言。`StoryShot` 还把镜头行为封装成 `is_dialogue()`、`get_tts_text()` 等方法，使数据对象本身成为模块间的稳定接口——这与 [[concepts/agent-orchestration-patterns|Agent 编排模式]] 中"用结构化消息替代自由文本传递"的思路一脉相承。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

### 风险前置：在 prompt 阶段消化生成模型的不稳定性

原文有两处容易被忽略的工程细节，本质上都是在 LLM 阶段就消化掉下游模型的已知缺陷：其一，自动识别用户文本中的"30 秒""1 分钟""90s"等时长表述，反推分镜数量与单镜时长；其二，把 `pan_left`/`pan_right` 这类高风险运镜自动替换为 `static`/`zoom_in`/`zoom_out`，避免主体在视频生成中移出画面。这两条规则意味着团队已经把视频生成模型的失败模式沉淀成了 prompt 层的约束——不稳定的模型能力被"前置消毒"，而不是靠后期修补。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

### 路由式生成：一致性由链路选择保证，而非模型保证

Stage 2 和 Stage 4 体现同一个设计哲学：按镜头属性路由到不同的生成链路。有 `character_id` 且有人物参考图的镜头走 Qwen Image Edit 图生图，纯场景镜头走 Z-image 文生图；`dialogue` 镜头走 MultiTalk lip-sync，其余走 Wan2.2 image-to-video。角色一致性因此不依赖单一模型的记忆能力，而是由"带参考图的图生图链路"结构性保证。双人镜头传第二张参考图、图像生成默认并发 2 以平衡 ComfyUI 资源压力，都是这条路由原则下的落地细节。这与 [[entities/ai-video-tools-third-stage-1779303117|AI 视频工具第三阶段]] 讨论的多模型分工趋势一致。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

### 后期合成层是对生成不确定性的兜底系统

原文直言 AIGC 视频片段"实际时长和目标时长不完全一致"是常态，因此 `video_editing.py` 成了整个系统可用性的关键：合并音轨时短则补静音、长则截断，按 `shot_durations` 修正每段长度以消除累计漂移，再统一烧录 SRT 字幕（必要时先执行可选的 Stage 4.5 分镜内嵌字幕清理，避免字幕重复）。可以把这一层理解为：前面四个生成阶段允许各自不完美，只要误差有界，最终由确定性代码收敛成可播放的成片。^[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline.md]

## 实践启示

1. **先定数据协议，再写 pipeline**。让 LLM 输出严格 JSON 的结构化 schema（字段互斥、枚举值明确），下游每个模块只读字段做分支——这是把"创意生成"变成"可编排系统"的第一块基石。
2. **把生成模型的已知失败模式翻译成 prompt 约束**。例如高漂移风险的运镜直接在脚本生成阶段替换掉，比事后重跑便宜得多；类似的"前置消毒"思路适用于任何带生成环节的自动化链路。
3. **一致性靠链路设计而不是模型选择**。需要角色/风格一致性的场景，优先走"参考图 + 图生图"路由；把文生图留给纯场景镜头，而不是指望单一模型同时兼顾灵活性和一致性。
4. **为每个阶段准备确定性的兜底层**。生成模型输出有界但不可控，最后必须有一层纯代码（音轨补齐/截断、时长校正、字幕烧录）把误差收敛掉，成片才可用。
5. **服务拆分按扩容维度切，不按功能切**。Gradio UI、ComfyUI、Fish-Speech 拆成三个服务，是因为 GPU 生图与 TTS 的扩容节奏和故障域不同；先单独验证各服务 workflow 可跑通，再接入 pipeline，问题定位成本最低。
6. **用参考图输入降低用户提示词负担**。Stage 0 的图像分析把风格/人物特征转成 prompt context 注入脚本生成，用户不写风格提示词也能得到贴合素材的结果——"让模型从素材本身继承约束"是降低 AIGC 工具使用门槛的有效模式。

---

→ [[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline|原文存档]]
