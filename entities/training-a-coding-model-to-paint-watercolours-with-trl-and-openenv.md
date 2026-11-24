---
title: "Training a Coding Model to Paint Watercolours with TRL and OpenEnv"
created: 2026-09-24
updated: 2026-11-23
type: entity
tags: [rl, post-training, trl, openenv, huggingface, coding-agent, multimodal]
sources: [raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv]
confidence: 0.7
provenance_state: extracted
---

# Training a Coding Model to Paint Watercolours with TRL and OpenEnv

## 核心内容

Hugging Face 官方博客展示了一条**用 RL 让代码生成模型习得视觉/绘图能力**的完整可复现路径：不修改模型权重结构，纯靠 post-training 阶段的 reward 信号把 coding model 变成能画水彩画的生成器。核心工具链是 TRL（训练）+ OpenEnv（环境接口），全流程工件（数据集、环境定义、训练配置）开源可复现。^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md]

## 方法要点

- **环境即接口**：OpenEnv 把"画得像不像"封装成可程序化评分的 Gym 式环境，模型每轮生成绘图代码 → 环境渲染出图 → 评分回传 reward。这让视觉质量这类模糊目标变成 RL 可优化的标量信号。^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md]
- **代码生成作为行动空间**：模型不直接输出像素，而是输出调用绘图库的代码；环境执行代码产出图像。这与 tool-use agent 的行动空间设计一致，可迁移到任意"代码驱动具象输出"的场景（前端 UI、3D 场景、数据可视化）。^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md]
- **Reward 设计**：视觉相似度评分（对比目标水彩画）+ 代码可执行性（语法/运行时错误惩罚）组合。RL 训练后模型从随机涂鸦收敛到能复现目标风格。^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md]
- **全流程开源工件**：数据集构建脚本、OpenEnv 环境定义、TRL 训练配置全部公开，属"复现级"教程而非纯概念文。^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md]

## 与 wiki 现有 RL 线索的关联

- 与 [[entities/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct]] 同属 TRL/GRPO 工具链，但本文的 reward 是**视觉相似度**而非结构化输出约束——展示了同一训练栈在不同输出模态上的复用。
- 环境封装思路与 [[entities/jitrl-just-in-time-reinforcement-learning-icml-2026-spotlight]] 的 RL 环境可得性问题互补：OpenEnv 提供了把任意程序化任务标准化的接口层。
- 对 agent harness 设计的启示：reward 环境即"可执行验证器"，与 [[entities/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026]] 中小米用可验证 RL 任务（代码/数学）驱动模型提升的路线一致。

## 深度分析

### TRL GRPO 训练环的力学：救活曲线的是四个参数，不是 reward

反直觉调试记录：三条 reward 实验全平（换池、去渲染噪声、关 judge 均无效），真正解锁训练的是 `GRPOTrainer` 四处配置^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:238-259]：

- **学习率 2e-5→5e-5**："LoRA Without Regret" 给 GRPO+LoRA 的上限，过低正是平线根因^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:254]。
- **scheduler linear→constant_with_warmup**：线性衰减到中段就花光学习率预算^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:255]。
- **`scale_rewards` group→none**：组内归一化下，一条 gate 拒绝的 0 分 rollout 会压缩同组其余 7 条优势——GRPO 特有的隐蔽失败模式^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:256]。
- **`target_modules`→`all-linear`**：dense 默认清单在 MoE 上因投影命名不同静默失效（只命中 10/40 层）；路由专家连 `all-linear` 也冻结，覆盖其余层已足够^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:248-257]。

与 [[concepts/grpo-policy-optimization-2026]] 互为注脚：训练器配置错误与 reward 错误不可区分，须用控制实验分离。

### OpenEnv 环境即验证器：比 Gym 接口更厚的封装

这里的"环境"把动作空间限制、渲染器、反作弊门控全部打包^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:156-158]：

1. **动作空间收紧**：p5.brush 暴露 47 个方法，prompt 只允许 10 个——长 API 参考会诱导模型发明不存在的方法，严格 allowlist 反而更稳^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:175]^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:183-189]。
2. **渲染即环境主体**：每步 15-18 分钟里 70-80% 花在无 GPU Space 的 Chromium 软件渲染（单次 69-96 秒，死线 90 秒）^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:404-414]——生成 150 行代码很便宜，执行它很贵。
3. **Gate 硬校验**：编译通过、真颜料上布、禁止画字骗评委^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:196]，与 [[concepts/verifier-driven-development]] 验证器优先思路同构。

两个暗坑：基建失败（渲染超时、scorer 无响应）曾以 0.0 计入 reward（占比 1.5-5.2%），把噪声当训练信号，修复为返回 None 并从组剔除^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:372-376]；OpenEnv 客户端 websocket 断连后仍被缓存，后续调用全失败（烧两个 run 才定位，修复已上游）^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:378-383]。与 [[entities/agentenv-agentic-rl-execution-environment]] 一致：执行环境可用性是训练稳定性的上限。

### 模糊视觉目标的 reward 分解：双评委分工 + 人工池即品味

Reward 四项：gate 0.05、长度 0.05、pairwise judge 0.60、HPSv3 0.30^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:112-117]。两个模型评委各答一问：HPSv3（7B 人类偏好模型，大众平均品味）判断"是不是一朵花"；pairwise judge（Qwen3-VL 对照参考池双向比较取胜率）判断"是否符合我选的风格"^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:119-128]^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:233-234]——通用审美先验与个人品味代理分层叠加。

典型的 "RL over taste"：无可验证正确答案，比 RLVR 更接近 RLHF^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:105-108]。品味不在超参里，而在 178 张人工评级的参考池^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:201-205]。与 [[concepts/rlvr-reinforcement-learning-verified-reasoning]] 对照，"验证器"换成了"品味代理"，是 [[entities/rl-beyond-the-verifiable-tanayj]] 的具体案例。两个细节：**分层课程藏在抽样里**——每步抽 4 张参考 love/okay 各半、胜过任何档同分，弱早期策略总有对手可赢（对原配方的刻意修改）^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:221-225]；**换池即换 reward**——目标自动改变而不改一行代码，reward 定义从写代码变成策展^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:231]。

### 策略到底学到了什么：分布中位数与无权重指令的失败

- **第一学习是"别画烂画"**：reward<0.3 的 rollout 在 judge-led 从 99 降到 16、hps-led 从 37 到 4；hps-only 组均值涨幅的 3/4 来自烂画变稀有。中位数 +0.155 而最佳仅 +0.034^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:289-323]——学习发生在分布中段，看每步最佳会低估进展。
- **judge 改变的是顶部**：纯 HPSv3 会饱和（理论上限 0.771）；有可击败的参考后顶部质量 +0.12/+0.16，颜料覆盖率翻倍。
- **无 reward 权重的指令是死字**：prompt 要求 15-30 个形状，策略实际用 7-9 个，n_shapes 与 reward 相关性≈0^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:316-319]。不被奖励服从，就不服从。
- **多样性是 reward 集的性质**：单一主题池让组内 rollout 收敛成"同一朵花的多次拍摄"，池决定什么算多样性，正如它决定质量^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:325-337]。与 [[entities/rlvr-entropy-collapse-steer-acl-2026-outstanding]] 熵坍缩讨论同源：扩多样性只能靠扩池。

## 实践启示

1. **训练不动，先跑无评委控制任务分离变量**：平线根因多在 trainer 配置而非 reward 设计；先确认极简任务曲线能起飞，再逐项加回难点^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:238-241]。
2. **MoE + LoRA 必须 `all-linear` 并核对覆盖层数**：dense 默认清单在 MoE 上静默失效（10/40 层）且不报错——"没学动"先查适配器覆盖面^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:248]。
3. **基建失败绝不能计 0 进 reward**：超时/掉线一律返回 None 并剔除——0 分是"画得烂"，None 才是"没发生"^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:372-376]。
4. **读曲线要区分"难抽签"与"策略退化"**：judge 每步重抽参考导致步间基准漂移，GRPO 组内 advantage 抗此噪而外部曲线不抗^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:390-396]。
5. **想让策略遵守的约束必须进 gate 或 reward**：无权重指令被策略"正确地"忽略——硬约束走 gate，软偏好走权重^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:316-319]。
6. **品味型 RL 的瓶颈与护城河都在池**：178 张模型生成的画定义了模型审美，池是没有原则性答案的部分^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:461-463]。下一步首选：给策略多步视觉反馈（当前"闭眼作画"，参考池本身却是三轮视觉批评的产物）、降模型规模（4B 已能写过 gate 草图，成本降一个量级）^[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv.md:435-445]。

## 评分依据

ark(glm-5.3-flash) 评分 v=7 c=7 stars=3（vxc=49 过线）：工程深度好、全流程工件可复现，但属"复现而非原创洞察"，故 value=7 而非 8+。

→ [[raw/articles/training-a-coding-model-to-paint-watercolours-with-trl-and-openenv|原文存档]]
