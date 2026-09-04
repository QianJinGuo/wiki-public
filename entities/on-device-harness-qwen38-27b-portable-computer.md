---

title: "端侧模型专用Harness：Qwen3.8-27B + Perplexity Portable Computer"
created: 2026-08-30
updated: 2026-09-05
type: entity
tags: [harness, on-device, local-inference, qwen, agent, privacy]
sources: [raw/articles/on-device-harness-qwen38-27b-portable-computer]
confidence: 0.75
---

# 端侧模型专用Harness：Qwen3.8-27B + Perplexity Portable Computer

Perplexity 推出的 Portable Computer 是一种端侧 Agent 运行范式：整个技术栈（模型、框架、对话、轨迹）默认在本地运行，仅在必要时调用外部服务。 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

## 核心设计

- **Local-first 架构**：模型推理、框架执行、会话历史全部在设备端完成，敏感数据不离开设备
- **框架-模型协同设计**：通用框架假定前沿模型能力（长上下文、大量工具、长链规划），本地小模型无法满足；Portable Computer 让框架根据模型能力量身定制，模型经过后训练以适配框架
- **零推理费用**：本地模型不产生 API 调用成本，仅在需要网络搜索/连接器/升级到云端顾问模型时才外部调用

## 技术栈

- 模型：Qwen 3.8 27B（270亿参数，可本地部署）
- 硬件：NVIDIA DGX Spark、Mac Studio M5 Ultra 等本地推理平台
- 框架：Perplexity 自研，针对小模型能力特性优化

## 与云端 Harness 的对比

| 维度 | 云端 Harness（Fable/Sol/Kimi K3） | 端侧 Harness（Portable Computer） |
|------|-----------------------------------|------------------------------------|
| 推理成本 | API 调用费 | 零（本地推理） |
| 数据隐私 | 数据离开设备 | 数据不离开设备 |
| 模型能力 | 前沿大模型 | 本地 27B 模型 |
| 功能完整性 | 全功能 | 部分功能需外部调用 |

## 启示

端侧 Harness 的核心洞察：不要让小模型管理为大模型设计的框架。框架应根据模型能力特性量身定制，而非期望小模型达到前沿模型的能力水平。这与 Harness Engineering 中"框架适配模型"的设计哲学一致。 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

→ [[raw/articles/on-device-harness-qwen38-27b-portable-computer|原文存档]] ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]