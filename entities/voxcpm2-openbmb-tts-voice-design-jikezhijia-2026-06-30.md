---
title: "VoxCPM2：OpenBMB 开源 Tokenizer-free TTS，Voice Design 文字描述生成声音"
created: 2026-06-30
updated: 2026-08-01
type: entity
tags: [tts, voice-synthesis, openbmb, voxcpm, tokenizer-free, voice-design, voice-cloning, diffusion, autoregressive, audio-vae, minicpm, open-source, apache-2.0, multi-language, chinese-dialects]
sources: [raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30]
confidence: 0.8
provenance_state: extracted
---

# VoxCPM2：OpenBMB 开源 Tokenizer-free TTS，Voice Design 文字描述生成声音

OpenBMB（面壁智能 / 清华实验室）在 2026 年 4 月开源的 TTS 模型 VoxCPM2，2B 参数，Apache 2.0 协议，不到三个月 GitHub 3.2 万 Star。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]

## 技术路线：Tokenizer-free 语音合成

与主流 TTS 路线的核心差异：不走离散 token 路线。主流方案将语音切为离散 token → 语言模型预测 → 声码器合成，但 token 压缩丢失细节。VoxCPM 在连续空间做扩散自回归生成。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]

**四阶段流水线**：Local Encoder → Text-Semantic LM → Residual Acoustic LM → Local DiT → AudioVAE V2 解码为 48kHz 波形。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]


底层 LM 基于 MiniCPM-4，扩散部分借鉴 DiTAR，流匹配参考 CosyVoice，AudioVAE 骨架来自 DAC。训练数据：200 万小时多语种语音。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]


## 三种生成模式

### 1. Voice Design（核心创新）
写文字描述直接生成声音，无需参考音频。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]
- 控制维度：性别、年龄、音色、情绪、语速、口音
- ElevenLabs 有类似付费功能，VoxCPM2 免费
- InstructTTSEval：中文三项与 Qwen3TTS 并列第一，英文三项压过 Hume 和 Mimo-Audio
- 当前局限：不是每次都稳定出理想效果，可能需要多次生成

### 2. 可控声音克隆
音色从参考音频提取，风格用文字指令控制——将传统声音克隆的音色+风格耦合拆解为独立控制。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]

### 3. 高保真克隆（Ultimate Cloning）
参考音频 + 准确文本转录 → 完整复刻音色、节奏和停顿。不控风格，追求一模一样。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]

## 多语言与方言

- 30 种语言：中英日韩法德西俄阿拉伯等，自动检测语种
- 9 种汉语方言：四川话、粤语、吴语、东北话、河南话、陕西话、山东话、天津话、闽南话

## 性能指标

| 指标 | 数值 |
|------|------|
| 输出采样率 | 48kHz（AudioVAE V2 非对称编解码：编码 16kHz → 解码 48kHz） |
| RTF（RTX 4090） | ~0.30（标准 PyTorch），~0.13（Nano-vLLM 加速） |
| 最低显存 | 8GB |
| Seed-TTS-eval WER | 1.84 |
| Seed-TTS-eval SIM | 75.3% |
| 30 语言 ASR 平均词错率 | 1.68% |

## 微调与社区生态

支持 SFT 全量微调和 LoRA 微调，5-10 分钟说话人音频即可适配。^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]

社区部署方案：
- **Nano-vLLM / vLLM-Omni**：高性能部署，OpenAI 兼容 API，支持并发
- **VoxCPM.cpp**：GGUF 量化，CPU 可跑
- **VoxCPM-ONNX**：ONNX 导出
- **VoxCPMANE**：Apple Neural Engine 后端
- **ComfyUI 节点插件**（3 个）
- **Rust 重写版**

## 快速上手

```bash
pip install voxcpm
```

```python
from voxcpm import VoxCPM
import soundfile as sf
model = VoxCPM.from_pretrained("openbmb/VoxCPM2", load_denoiser=False)
wav = model.generate(text="VoxCPM2 是目前推荐的多语言语音合成版本。", cfg_value=2.0, inference_timesteps=10)
sf.write("demo.wav", wav, model.tts_model.sample_rate)
```

CLI：`voxcpm design --text "你好世界" --output out.wav`^[raw/articles/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30.md]


## 相关实体

- [[entities/pilotdeck-agent-os-openbmb-tsinghua|PilotDeck：清华系Agent操作系统]] — 同团队（OpenBMB/面壁智能/清华）的 Agent OS 项目
- [[entities/edgeclaw-openbmb|EdgeClaw — 端云两栖龙虾框架]] — 同团队的开源框架

## References

- GitHub: https://github.com/OpenBMB/VoxCPM
