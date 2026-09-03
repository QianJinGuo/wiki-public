---
title: "Story2Video: 从故事文本到可发布短视频的 AIGC Pipeline"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [aigc, video-generation, pipeline, ai]
sources: [raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline]
confidence: 0.8
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

---

→ [[raw/articles/story2video-技术实践从故事文本到可发布短视频的-aigc-pipeline|原文存档]]
