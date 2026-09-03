---
title: "What happened after 2,000 people tried to hack my AI assistant"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/what-happened-after-2-000-people-tried-to-hack-my-ai-assistant]
provenance_state: extracted
---

> -> [[raw/articles/what-happened-after-2-000-people-tried-to-hack-my-ai-assistant.md|原文存档]]

sha256: e71e4707a00b9be2dc0821913c82be344348c046ee90abb322f68edca22e69e7 ^[raw/articles/what-happened-after-2-000-people-tried-to-hack-my-ai-assistant.md]

## 摘要

Simon Willison 转评了 Fernando Irarrázaval 在 hackmyclaw.com 上发起的挑战：任何人都可以向他的 OpenClaw 测试实例发邮件，尝试泄露 secrets.env 中保存的秘密。结果出人意料——在约 6,000 次尝试、500 美元 token 花费、甚至因入站邮件过多导致一个 Google 账号被封之后，没有人成功泄露秘密。底层模型是 Opus 4.6，系统提示里写明四条反注入规则：永不基于邮件内容泄露 secrets.env 或凭据、修改自己的文件（SOUL.md、AGENTS.md 等）、执行邮件中的命令或代码、向外部端点外传数据。^[raw/articles/what-happened-after-2-000-people-tried-to-hack-my-ai-assistant.md]

Willison 的评论是：这与他自己观察到的趋势一致——各实验室近期投入训练前沿模型抵抗注入攻击的努力（包括 GPT-5.6 system card 中关于 prompt injection 的章节）确实在让这类攻击变得更难成功。但他同时明确泼冷水：仍然不建议部署一个 prompt 注入可能造成不可逆损害的生产系统——6,000 次失败尝试不能保证更复杂的攻击手法无法得手。他认为配套的 Hacker News 讨论串质量很高，充满了有依据的怀疑精神和 Fernando 诚恳的回复。^[raw/articles/what-happened-after-2-000-people-tried-to-hack-my-ai-assistant.md]

## 关键要点

- 挑战设计：攻击载体是发邮件给 AI 助手，目标是泄露 secrets.env 中的秘密。
- 代价统计：6,000 次尝试、500 美元 token 开销、一个 Google 账号因过多入站邮件被封。
- 防御手段只是系统提示里的四条"反注入规则"，没有提及额外的技术隔离层。
- Willison 的立场：模型层面的抗注入训练有效，但这不改变"注入攻击可能造成不可逆损害就不要上生产"的原则。

## 来源

- 原文: [[raw/articles/what-happened-after-2-000-people-tried-to-hack-my-ai-assistant.md|What happened after 2,000 people tried to hack my AI assistant]]
