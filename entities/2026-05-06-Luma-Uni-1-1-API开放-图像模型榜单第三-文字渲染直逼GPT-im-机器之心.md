---
title: "Luma Uni 1 1 API开放 图像模型榜单第三 文字渲染直逼GPT im 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-06-Luma-Uni-1-1-API开放-图像模型榜单第三-文字渲染直逼GPT-im-机器之心]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-06-Luma-Uni-1-1-API开放-图像模型榜单第三-文字渲染直逼GPT-im-机器之心.md|原文存档]]

sha256: 9296fc93995d77c2c79dc941f6470b97d99f1a16169b607cdcb58cdd479217b1 ^[raw/articles/2026-05-06-Luma-Uni-1-1-API开放-图像模型榜单第三-文字渲染直逼GPT-im-机器之心.md]

## 摘要

Luma 将统一图像模型 Uni-1 升级到 1.1 并开放 API：在第三方盲测平台 LMArena 的图像生成榜单上，Uni-1.1 与 Uni-1.1-Max 进入实验室榜前三，仅次于 OpenAI 和 Google，排在 Microsoft AI、xAI、Reve、阿里、Black Forest Labs、腾讯与字节之前；单图最低 0.0404 美元，价格与延迟均不到同类模型一半。技术路线是 decoder-only 自回归 Transformer，把文本 token 与图像 token 放在同一交错序列同时建模，"先把意图想清楚，再让像素落下来"，API 拆成 Reasoning 端点（指令解构、构图规划、品牌/角色/产品约束锁定）与 Generation 端点（像素级渲染）；9 张参考图作为模型层级硬约束直接进入主序列，而非事后 LoRA/IP-Adapter，多轮按句编辑跨轮保留主体身份与空间关系。文章用四组生成案例展示能力（单图直出整页 2036 新闻网站、Sagittarius A 蓝图风格信息图、1957-2025 火箭同比例对比插画、含十二张同人设缩略图的中文海报"水·韵"）。商业侧：Adidas、Mazda、Publicis Groupe、Serviceplan 等品牌接入；某品牌原预算约 1500 万美元、周期一年的广告活动经 Uni-1.1 工作流在约 40 小时内、以低于 2 万美元成本完成多国本地化。核心团队不到 15 人，由宋佳铭（DDIM 作者）与沈博魁（CVPR 2018 Best Paper）两位华人学者领衔；路线图下一步从静态图像扩展到视频、语音与交互式世界模拟。^[raw/articles/2026-05-06-Luma-Uni-1-1-API开放-图像模型榜单第三-文字渲染直逼GPT-im-机器之心.md]

## 关键要点

- 榜单成绩：Uni-1.1 与 Uni-1.1-Max 进 LMArena 图像生成实验室榜前三（图像编辑第 3、文生图第 3），为第一代统一图像模型达到此成绩的首例
- 架构选择：与"理解用 CLIP/Florence/Grounding-DINO 编码器 + 生成用 Latent Diffusion/Rectified Flow"的分立主流不同，Uni-1.1 用 decoder-only 自回归 Transformer 对文本与图像 token 同序列建模，字符级控制、多参考图与多轮编辑状态保持由模型内部能力驱动
- Luma 公开经验：生成训练能显著提升模型细粒度理解能力（学会"怎么画"后"看懂"也变强），与认知科学"生成式心智模型"假说呼应，是选择统一架构的重要动机
- 生产级能力：单次调用最多 9 张参考图联合输入（品牌主形象、产品照、面料样、logo 等作硬约束）；按句编辑如"去掉前面这只熊""整体改成黑白照片"且默认保留其他元素
- 团队与人物：核心研究团队不到 15 人；首席科学家宋佳铭（清华本科、斯坦福博士，DDIM 把扩散采样从数百上千步压到数十步）；Uni 系列研究负责人沈博魁（斯坦福本博，CVPR 2018 Best Paper、RSS 2022 Best Student Paper）
- API 与定价：Build（按量）与 Scale（预留吞吐，8 单元起订）两档；SDK 覆盖 Python、JavaScript/TypeScript、Go 与 CLI，接入入口 platform.lumalabs.ai

## 来源

- 原文: [[raw/articles/2026-05-06-Luma-Uni-1-1-API开放-图像模型榜单第三-文字渲染直逼GPT-im-机器之心.md|Luma Uni 1 1 API开放 图像模型榜单第三 文字渲染直逼GPT im 机器之心]]
