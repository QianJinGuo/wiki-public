---

title: "Teaching Everyone to Fish for Tokens：开源 AI 生态的 token 经济学"
type: entity
created: 2026-08-30
updated: 2026-09-17
tags: [open-source, llm, inference, economics, nvidia, meta, post-training, ecosystem]
sources:
  - raw/articles/teaching-everyone-to-fish-for-tokens
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Teaching Everyone to Fish for Tokens：开源 AI 生态的 token 经济学

Nathan Lambert（Interconnects AI）分析了开源 AI 生态的经济可持续性问题。核心观点：开源模型生态正面临**存在性窗口期**——如果 Nvidia 的开源投资不能在几年内产生回报，或者没有其他开源模型公司建立起平台级财务反馈循环，开源生态将被迫转向效率、可修改性、专业化等差异化路径。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

## 两种未来

### 未来一：开源"奏效"
Nvidia 投入 260 亿美元推动开源模型，目标是创造远超模型构建成本的芯片需求。如果成功，将形成"教所有人制造 token 机器"的生态，智能不会被垄断。关键假设：
- 开源 recipe 对 Nvidia 有效
- 推理需求覆盖训练成本
- 社区贡献改进数据/训练代码 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

### 未来二：开源分叉
如果财务正循环无法建立，开源模型将与前沿闭源模型分叉：
- **开源方向**：效率、可修改性、专业化（企业私有数据、重复业务任务）
- **闭源方向**：知识工作协作、药物发现、软件工程等高价值领域 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

Lambert 认为这是**最可能的结果**——开源模型仍有用，但在长尾生态系统中填补空白。

## Post-Training 的演变

当前开源生态由**后训练**（post-training）兴趣爆发推动：开发者使用 DeepSeek V4 Flash、Inkling Small、GLM 5.X 等模型进行任务特定微调（如 Tinker 平台）。 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

但训练正在变得**更复杂、更抽象化**：
- 基础模型训练的经济门槛持续提高
- 能够训练基础模型的开源构建者数量在减少
- 开源模型开始实验**收入分成许可证**（如 Kimi K3、Qwen3.8） ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

Lambert 预测训练 lexicon 将演变：`pretraining → reasoning training → post-training` ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

## Nvidia vs Meta 的不同策略

| | Nvidia | Meta |
|---|---|---|
| **策略** | 教所有人制造 token 机器 | 用开放权重冲垮竞争对手 |
| **目标** | 创造芯片需求 | 削弱 Anthropic/OpenAI 的 token 销售收入 |
| **方式** | 开源训练数据+代码 | 开放模型权重（如 Muse Spark 1.2） |
| **经济逻辑** | 推理需求 → 芯片销售 | 商品化互补品 → 竞争对手收入下降 |

两者都在**商品化自己的互补品**，但方式不同。

## 关键信号

1. **退出者**：Databricks、01.ai 等公司退出基础模型训练
2. **许可证实验**：Kimi K3、Qwen3.8 的收入分成模式
3. **后训练繁荣**：微调 API（Tinker）的流行
4. **Nvidia 投资规模**：260 亿美元的开源赌注 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

## 深度分析

### 决定两种未来的不是模型质量，而是资本回报闭环

"开源是否奏效"表面上是技术问题，实质是**资本回收率**问题：Nvidia 投入约 260 亿美元做近乎开源的 recipe，它买的不是模型本身，而是"把推理需求转化为芯片订单"这条路径的持续性。判据因此不是开源模型是否追平前沿，而是**开源诱导出的推理需求能否以接近 Anthropic/OpenAI API 利润增速的斜率回流到出资方**。这也是 Lambert 判断第二种未来（开源与闭源分叉）最可能的原因：一旦回报斜率低于闭源 API 的利润斜率，开源侧就无法在数十年尺度上"保持同步"，只能退守效率、可修改性与专业化。未来的分岔点由现金流而非 benchmark 决定——这是理解整篇论证的枢纽，也决定了 [[concepts/open-source-ai-ecosystem|开源 AI 生态]] 是走向自我维持还是长期依赖单一出资方。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

### Post-Training 的抽象化 = 价值链利润的重新定位

把 `pretraining → reasoning training → post-training` 读成一条价值链，会看到利润在下移而非消失：**预训练**是极高资本门槛、极少数玩家的环节；**reasoning training**（把基座训练成通用 agentic reasoner）正变得像几年前的规模化预训练一样不透明，从而成为新的守门环节；**后训练**则向数据、任务特化、评测与部署侧开放。利润于是从"造模型"迁移到"掌握私有数据与任务定义权"的位置——这正是 Tinker 一类微调 API 繁荣的结构性原因：它们卖的不是模型能力，而是把通用智能转成特定业务行为的转换成本。反过来说，**基座模型正在成为可替换的投入品**，而护城河落在数据回流与部署运维上，[[concepts/inference-optimization|推理优化]] 这类工程能力因此从成本项变成差异化来源。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

### Nvidia 与 Meta：同一条曲线上的两种下注

两家都在商品化自己的互补品，但**收益结构相反**。Nvidia 的互补品是"别人模型上的推理需求"：开放权重越强越便宜，芯片与配套需求越大，这是一笔正和、且需要生态自我维持的下注，所以它必须关心开源能否自养。Meta 的互补品是竞争对手的 token 销售收入：开放权重只要足够强、足够便宜，就能压低 Anthropic/OpenAI 的定价与增长斜率（[[entities/meta-muse-spark-11-agentic-coding-model-2026|Muse Spark]] 一类模型的开放权重效果即在此），而 Meta 的收益来自自身业务与人才市场，不需要开源生态自己赚钱。同一个动作（释放权重），一个是**需求创造**，一个是**防御性护城河**。差别在耐久性：Nvidia 有动机长期出资并公开可公开的数据与训练代码，Meta 只需"洪水漫灌"式的低价供给延续。看谁在出钱、动机是什么，比看谁放出的权重更强更能预测生态走向。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

### 收入分成许可证的激励相容难题

[[entities/kimi-k3-the-open-weights-escalation|Kimi K3]] 与 [[entities/qwen38-max-first-open-weights-release|Qwen3.8]] 试水下游产品与推理的收入分成许可证，本质是在"开放权重"与"可融资"之间寻找中间态，但存在难以回避的激励不相容：**收入分成恰好对"我们最希望发生的行为"（大规模部署）收费**。它带来三重成本——下游集成方的合规与追溯负担、执行能力的上限（小团队难以全球执法）、以及法律不确定性对采购决策的抑制。更关键的是，带分成的许可证已不满足开源定义，于是同时失去"开源"的信任红利与"闭源"的定价能力，除非模型强度已接近前沿、使分成成为可接受的门票。这批实验的成败因此直接决定 Nvidia 需求增长战略能否延续：**需要成功的是这些开放模型公司，而不是 Nvidia 自己**。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

### 推理趋近免费、权重免费之后，价值捕获点跑到哪里

当权重免费、推理成本持续下探，token 本身不再有定价权，价值捕获必然转移到三处：**分发**（谁掌握终端用户与企业采购关系，把模型打包进既有云与办公套件）、**专有数据闭环**（私有语料 + 持续后训练的复利）、以及**切换成本**（harness、评测、运维、合规）。这也解释了长尾化的具体形态：企业私有数据、可 on-prem 运行的重复业务任务天然需要可修改性与数据不出域，是开放权重唯一有定价权的地带；知识工作协作、药物发现、软件工程等高价值区仍由闭源垄断。对生态中其他玩家，结论残酷但清晰：**开放权重创造选择权，但不自动创造利润**，利润属于在权重之上建立了分发或数据复利的人——这与 [[entities/how-open-model-ecosystems-compound|开放模型生态的复利效应]] 所描述的机制一致，也是 [[entities/25-the-unbearable-cheapness-of-open-weight-models|开放权重模型的廉价化]] 的另一面。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

## 实践启示

1. **别把模型能力当估值锚点，把模型层当可替换投入品来架构。** 在应用层与模型层之间保留抽象（统一 API、路由、可切换的多模型后端），让换基座的成本保持在"改配置"级别；任何"只有某家模型能跑"的决策都应视为需主动支付溢价的负债。

2. **成本模型要按推理成本下降曲线写，并把"权重免费"设为基准情景。** 若单位经济成立的前提是 token 价格维持高位，那是在对补贴定价做多；应同时准备"推理趋近免费、毛利转向数据/运维/合规"的第二套模型。

3. **许可证先过法务，再做技术选型。** 明确区分真开源与收入分成/使用限制型许可。后者对已规模化变现的业务意味着收入敞口与审计负担，对早期产品则是"先上车后付费"的期权——取决于预计部署规模。

4. **把不可替代的资产押在私有数据与任务特化上，而不是基座版本上。** 用后训练把通用智能固化为业务流程（评测集、领域数据、工具链），这类资产换模型时不归零：基座升级应是"重跑一遍微调"，而非"重做一遍产品"。

5. **在多供应商之间保持"可信切换"而非"多签合同"。** 真正降低 lock-in 的是随时可切换的技术能力（权重自托管路径、推理框架、降级方案）与团队经验；合规敏感场景应提前验证 on-prem/自托管路线确实可行。

6. **把三个信号设为决策触发器：** ①开放模型公司退出基座训练是否从"异常值"变成常态；②收入分成许可证是否被主流下游规模化接受；③Nvidia 的开源出资是否延续或加码。三者共同决定开源侧走向自我维持还是长尾分叉——模型栈策略应对两种情形都留余量。

→ [[raw/articles/teaching-everyone-to-fish-for-tokens|原文存档]] ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]