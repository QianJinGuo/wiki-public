---
title: "1. 准备 CLI（二选一）curl -fsSL https://nacos.io/nacos-installer.sh | bash -s -- --cli# 或者直接使用 npxnpx @nacos-group/cli@latest skill-sync --help# 2. 配置 CLI profilenpx @nacos-group/cli@latest profile edit test"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-24-别再手动复制-Skill-了-多-Agent-时代的-Skill-管理方案-阿里云云原生]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-24-别再手动复制-Skill-了-多-Agent-时代的-Skill-管理方案-阿里云云原生.md|原文存档]]

sha256: 6194a420b01755a64083dc77aa59abf1bddac90dccdbb55717357cdf39993144 ^[raw/articles/2026-06-24-别再手动复制-Skill-了-多-Agent-时代的-Skill-管理方案-阿里云云原生.md]

## 摘要

阿里云云原生团队发布 Nacos Skill Sync，解决多 Agent 时代的 Skill 管理碎片化问题：同一份 Skill 在 Codex 里更新了、Claude Code 里还是旧版、Cursor 下还可能有同名异容副本，手动复制时间一长就陷入"哪份最新、该用谁覆盖谁"的混乱。此前社区方案的共同缺口是：Git submodule/Monorepo 太重、Syncthing 只懂文件不懂 Skill 语义且双向修改易覆盖、LangSmith 等 Prompt 平台不管本地 Agent 配置文件。Nacos Skill Sync 的一句话定位是"把 Skill 收敛到一个中心仓库，再按需分发给各个 Agent"——默认用软链接让各 Agent 目录指向中心仓库（改一处全部生效），环境不支持软链接时切换复制模式，同步状态随时可查。^[raw/articles/2026-06-24-别再手动复制-Skill-了-多-Agent-时代的-Skill-管理方案-阿里云云原生.md]

它提供两种模式：Local mode（本地中心仓库 + 软链接/复制同步 + 零服务依赖）会自动发现 Codex、Claude、Qoder、QoderWork、Cursor、Kiro、Lingma、CoPaw、OpenClaw 以及通用的 ~/.agents/skills 和 ~/.skills 目录，用 skill-sync add 可将散落各处的同名 Skill 逐步纳入，内容不一致时会停下让用户选择以哪份为准，remove 则先把中心仓库内容复制回各 Agent 再移除同步状态；Registry mode（Nacos AI Registry + 可视化管理 + 跨设备共享）提供版本治理（草稿、审核、发布、回滚、label）和双向流通，Skill 从"本地配置文件"变成可追溯的 Registry 资产。日常维护只需看 status 输出：Synced/Linked/Local changes/Uploaded/Conflict/Upload blocked 六种状态标明每个 Skill 覆盖了哪些 Agent、下一步该干什么；冲突时 resolve 命令默认保守、不会擅自替用户选择版本。推荐上手方式是把官方 SKILL.md 发给 Agent 自驱动执行同步。^[raw/articles/2026-06-24-别再手动复制-Skill-了-多-Agent-时代的-Skill-管理方案-阿里云云原生.md]

## 关键要点

- 文章给出的两个使用案例：个人本机用 Local mode 收拢"记录工作内容&生成周报"Skill（避免把私人工作记录同步出去），团队用 Registry mode 管理"文档统一格式"Skill（多设备共享同一套格式规范）。
- 切换同步来源时，CLI 会先把旧 profile 的软链接安全落回各 Agent 本地副本，不同 profile 的 Skill 不会互相覆盖。
- 核心理念："让 Skill 有一份可信来源。Agent 可以换，Skill 不应该跟着散。"
- Local mode 可自然升级到 Registry mode，绑定一个 profile 即可接入远端，无需重新整理。
- 相关仓库：Nacos 开源地址 github.com/alibaba/nacos，Nacos CLI github.com/nacos-group/nacos-cli。

## 来源

- 原文: [[raw/articles/2026-06-24-别再手动复制-Skill-了-多-Agent-时代的-Skill-管理方案-阿里云云原生.md|1. 准备 CLI（二选一）curl -fsSL https://nacos.io/nacos-installer.sh | bash -s -- --cli# 或者直接使用 npxnpx @nacos-group/cli@latest skill-sync --help# 2. 配置 CLI profilenpx @nacos-group/cli@latest profile edit test]]
