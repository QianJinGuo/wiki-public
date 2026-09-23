---

title: "【VLM】详解视觉语言模型原理及代码，以DeepSeek-VL为例"
type: entity
created: 2026-07-04
updated: 2026-09-23
tags: [wechat, ai]
rating: v7c7
sources:
  - raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 【VLM】详解视觉语言模型原理及代码，以DeepSeek-VL为例

**来源**: 炼钢AI

**发布日期**: 2024-08-15^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]


**原文链接**: https://mp.weixin.qq.com/s/jrwLNOnoS_9O3R0ECp7IvA ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

---

这是我在微信公众号的第一篇文章，其实之前也在zhi hu更新过一些文章（id：战士金），也会慢慢搬运过来。LLM、RAG、VLM啥的后续可能都会更新。目前业余时间预训练了一个1B的LLM，使用T级别的数据，欢迎关注，各种技术细节也都分享过：https://github.com/zhanshijinwat/Steel-LLM ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

最近开始看看视觉语言模型（VLM）相关的东西了，之前没特别仔细看过代码。翻了几篇比较知名的开源VLM技术报告，感觉DeepSeek-VL算是写的比较好的，因此本文就以DeepSeek-VL为例，结合代码写一写VLM的细节。VLM和LLM比较共性的东西比如Self Attention之类的本文就不过多介绍了，重点讲一讲VLM独有的内容。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

DeepSeek-VL github链接：https//github.com/deepseek-ai/DeepSeek-VL/tree/main ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

1

原理

1.1 模型训练

VLM通常分为3个部分：视觉编码编码器、视觉适配器和LLM。^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]


视觉编码器用于将图像转换为向量表示，在DeepSeek-VL中，图像被视觉编码器转换为576个向量（图像的token embedding）。VLM的视觉编码器 直接使其他模型预训练好的参数 ，普遍使用的视觉编码器结构为ViT（Vision Transformer），但可能是不同方式训练出来的，例如DeepSeek-VL使用的是Siglip和SAM训练出来的ViT，而Qwen-VL使用的是OpenCLIP的ViT。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

视觉适配器用于将用于将ViT的输出图像的token embeeding映射到与文本embedding相同的空间，便于让LLM理解图像中的内容。常见的结构有多层MLP、cross attention等。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

LLM是VLM的核心，视觉编码器和适配器最终产出的图像的token embedding都是要输入到LLM进行理解的，并由LLM输出关于图像的回答。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

DeepSeek-VL训练可分为3个阶段（不同VLM训练的阶段数和每个阶段里训练哪部分参数会有所不同），如下图所示： ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

阶段1 ：这一阶段的主要目标是将视觉和语言信息在embedding空间建立联系，从而促进LLM能对图像中的实体有所理解。该阶段只对视觉适配器部分的参数继续进行训练，LLM和视觉编码器的参数冻结。该阶段训练了从ShareGPT4V 获得的 125 万个图像-文本对，以及 250 万个从文档OCR出来的图像文本对。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

阶段2 ：该阶段主要目标是让LLM理解图像输入，保持图像编码器参数冻结，训练图像适配器以及LLM。如果单纯使用图像-文本对来训练LLM的话，会使LLM的语言能力下降，因此训练数据中也混合了纯文本数据，多模态数据：纯文本数据=7：3。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

阶段3 ：该阶段是指令微调阶段，增强模型的指令遵循与对话能力。该阶段对视觉编码器、视觉适配器和LLM进行联合训练。和LLM的SFT过程类似，输入的instruct部分不计算loss。和阶段2类似，除了多模态数据外，该阶段也使用了纯文本数据进行训练。 ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

1.2 模型结构

具体的模型结构方面，DeepSeek-VL的LLM部分使用的是自家的DeepSeek LL^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]


^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

→ [[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例|原文存档]] ^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md]

---

## 深度分析

### 模态桥接：VLM 的本质是给 LLM 加一个「翻译层」

把 VLM 拆开看，它不是一种新模型，而是三种现成组件的组合：视觉编码器把图像变成向量序列，视觉适配器把这些向量映射到文本 embedding 空间，LLM 负责最终的理解与生成^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:34-40]。文章点出一个容易被忽略的事实：VLM 和 LLM 在生成原理上没有任何不同，唯一的变化是输入序列里混入了图像 token^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:80]。也就是说，模态桥接全部发生在 embedding 空间的入口处——只要图像 token 能被投影到与文本 token 同一空间，[[concepts/transformer-architecture|Transformer]] 主体无需任何改动就能「读懂」图像。这与 [[entities/deepseek-v3-moe-architecture|DeepSeek V3]] 一脉的思路一致：架构创新尽量收敛到输入/输出边界，核心计算图保持通用。

### 混合视觉编码：全局语义与局部细节的双塔设计

DeepSeek-VL 在视觉侧最值得注意的选择是混合编码：SigLIP 的 ViT 接收 384×384 低分辨率图像提取粗粒度语义，SAM-B 的 ViT 接收 1024×1024 高分辨率图像捕捉细节，两路输出都被规整成 (576, 1024) 的 token 序列^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:54,123-137]。为什么不只用一个高分辨率模型？单塔方案要么牺牲细节（resize 到小图），要么让 token 数爆炸（大图切更多 patch），双塔用固定 576 个 token 同时保住两种粒度，是一个典型的「用算力换表达」权衡。融合环节则相当朴素：仅用一个 MLP 投影器，代码里预留了 feature 拼接、sequence 拼接、相加、tuple 直传四种策略，DeepSeek 实际选了最简单的 tuple 直传，把融合决策下放给下游模块^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:70,137-139]。

### 跨模态对齐训练：三阶段渐进解冻，纯文本数据保底

对齐训练与纯 LLM 预训练的根本差异在于：它要解决的是两个已训练好的表示空间之间的「分布匹配」，而不是从零学语言。DeepSeek-VL 分三阶段解冻：阶段 1 只训视觉适配器（LLM 与视觉编码器全冻结），用 125 万图文对加 250 万文档 OCR 对建立粗对齐；阶段 2 解冻 LLM 与适配器、冻结视觉塔；阶段 3 才做全参数联合 SFT^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:42-48]。这里有一条纯 LLM 训练不会遇到的坑：只用图文对训练会让 LLM 的语言能力退化，因此阶段 2、3 都混入纯文本数据，多模态与纯文本比例为 7:3^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:46-48]。这个「数据配比防退化」的思路对任何多模态微调都适用，与 [[concepts/llm-pretraining-vs-sft|预训练与 SFT 的数据分布差异]] 讨论的是同一类问题。

### 代码级要点：一个占位符如何膨胀成 576 个图像 token

代码层面最有信息量的是输入构造管线。对话里的 `<image_placeholder>` 在 token id 层面只占 1 个位置（id 100015），processor 会找到这些位置并复制成 576 个，图像才真正「占座」^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:123]。随后 `prepare_inputs_embeds` 统一提取文本与图像的 embedding：文本 token 从 embedding table 索引，占位符位置填入视觉塔输出，整个张量通过 `inputs_embeds` 参数直接喂给 LLM 的 `generate`——绕过了 token id 查表这一步^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:153-155]。三个配置文件各司其职：tokenizer 管文本、preprocessor 管图像数值预处理、processor 管占位符与 token 数量，这套分层对读懂任何开源 VLM 的推理代码都有迁移价值^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:82-111]。

## 实践启示

### 1. 选型判断：看输入里有没有「像素」

Agent 管线里选 VLM 还是纯 LLM，判断标准简单粗暴：任务输入是否包含图像、截图、扫描件或图表。纯文本任务用 VLM 是纯浪费——每张图固定膨胀 576 个 token，推高成本与延迟却不带来信息^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:36,123]。反过来，只要输入带像素，纯 LLM 在架构上就无能为力，因为图像根本进不了 embedding 空间。

### 2. 文档与图像密集型工作流是 VLM 的主场

阶段 1 训练数据里 250 万对来自文档 OCR，占了全部对齐数据的三分之二——DeepSeek-VL 从设计之初就把文档理解当作核心场景^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:44]。对 agent 工作流的直接含义：处理 PDF、报表截图、UI 截图、带图表的网页这类工作流，VLM 能把「先 OCR 再喂文本」的两段式管线压缩成端到端调用，少一层信息损耗。已在 [[entities/cost-effective-deployment-of-vision-language-models-for-pet-behavior-detection-o|VLM 行为检测部署案例]] 中落地的同类思路表明，垂直场景下这套架构的部署成本是可控的。

### 3. 集成视角：把 VLM 当「带视觉前端的 LLM」接入

对应用层来说，VLM 的集成接口和 LLM 几乎同构：conversation dict 只比纯文本多一个 `images` 字段，支持多图（多个占位符对应多张图），其余生成参数原样传递^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:80-111]。需要预留的两件事：一是上下文预算按每图 576 token 计算，多图输入很快吃掉窗口；二是图像预处理（resize、归一化、背景填充）由 processor 封装，自建管线时不要绕过它，否则数值分布会偏离训练分布^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:111-121]。

### 4. 微调多模态模型时，纯文本数据是必需品不是配菜

7:3 的数据配比值得写进任何多模态训练清单：图文数据会让 LLM 遗忘语言能力，纯文本数据是防止回退的保底手段^[raw/articles/vlm详解视觉语言模型原理及代码以deepseek-vl为例.md:46]。同理可推广到 agent 工具调用数据的混合训练——任何引入新 token 分布的微调，都应保留一定比例的原任务数据。

### 5. 感知对齐不等于感知正确

VLM 的训练目标是让图像 token 「进入」语言空间，但这只保证模型看得见，不保证看得对。[[entities/235b参数也没用港中文等发布7模态数据集专测顶级vlm的感知盲区|顶级 VLM 感知盲区评测]] 显示，参数量堆到 235B 仍存在系统性感知盲区。把 VLM 接入生产管线时，应针对业务图像类型专门构造感知评测集，而不是默认「模型这么大所以不会看错」。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

