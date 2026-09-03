---

title: AI芯片架构：从逻辑门到矩阵乘法
created: 2026-05-23
updated: 2026-08-29
type: entity
tags: [ai-chip, systolic-array, tpu, gpu, hardware, architecture, matrix-multiplication, fpga, asic, computer-architecture]
sources: [raw/articles/ai-chip-architecture-first-principles]
review_value: 7
confidence: 0.8
---

# AI芯片架构：从逻辑门到矩阵乘法

## 核心主题

芯片最底层的原语是逻辑门（与门、或门、非门），通过金属导线连接。AI芯片的核心任务是矩阵乘法，最基本操作是「乘加运算」（multiply-accumulate）：两个数相乘，再加到累加器里 ^[raw/articles/ai-chip-architecture-first-principles.md]。

## 乘法器：从比特到面积的代价

4比特乘4比特的乘法需要 p×q=16 个与门来产生所有部分积，真正的功夫却在求和阶段。求和用「全加器」（3→2压缩器）：输入三个单比特，输出两个比特（因为0+1+1=2，二进制是10）。Dadda乘法器把需要加的比特排成网格，用全加器反复压缩，直到剩下一个数。 ^[raw/articles/ai-chip-architecture-first-principles.md]

关键规律：**乘法器面积和比特位宽是平方关系**。精度减半，理论上算力增加4倍。NVIDIA在B300及之后已承认FP4性能是FP8的3倍（理论上4倍）。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## 数据搬运税：被忽视的成本结构

在传统CUDA核心或CPU核心里，乘法器旁边的寄存器堆（register file）负责存取操作数。从8个寄存器中随机读取一个需要多路选择器（mux）——8条目×4比特的寄存器堆，读取三个操作数需要3×8×4=96个与门。而乘加器本身只需要16个与门。 ^[raw/articles/ai-chip-architecture-first-principles.md]

> **核心矛盾：计算占的面积很小，数据搬运却花了最多的面积。** 在这个设计里，大约7/8的成本花在数据读写上，只有1/8花在真正的计算上。

## 脉动阵列：改变游戏规则的升维

Tensor Core背后的核心思想是「脉动阵列」（systolic array）。思路是往上走一层循环：把「一整段矩阵乘法循环」固化到硬件里，而非单独固化「一次乘加」。 ^[raw/articles/ai-chip-architecture-first-principles.md]

两个关键优化： ^[raw/articles/ai-chip-architecture-first-principles.md]
1. **单位数据搬运能做更多计算** ^[raw/articles/ai-chip-architecture-first-principles.md]
2. **权重矩阵长期保持不变**：权重存在本地寄存器里，被反复使用，不需要每次从寄存器堆重新加载 ^[raw/articles/ai-chip-architecture-first-principles.md]

权重矩阵用菊花链方式逐行加载，花y个时钟周期完成。因为权重停留时间长，加载慢一点没关系——关键是加载带宽被控制在 O(x)，而不是 O(xy)。 ^[raw/articles/ai-chip-architecture-first-principles.md]

> 这是一个贯穿全栈的主题：在芯片设计的每一个层级，从逻辑门到数据中心，都在做同一件事——**最大化计算相对于通信的比例**。

脉动阵列是目前已知最高效的矩阵乘法电路。老TPU里的脉动阵列是128×128的规模。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## 时钟周期：芯片的心跳

1000亿个晶体管并行工作，靠全局时钟信号同步——大约每纳秒一次，所有电路暂停、对齐、同时迈出下一步。 ^[raw/articles/ai-chip-architecture-first-principles.md]

时钟周期瓶颈在两个寄存器之间「逻辑云」的延迟。如果这段逻辑太复杂，信号来不及稳定就会出错。芯片设计师会留出裕量，确保在25%的时钟周期内完成。 ^[raw/articles/ai-chip-architecture-first-principles.md]

把逻辑云劈成两半，中间插流水线寄存器，时钟频率能翻倍——代价是多了存储面积。有反馈回路的情况（如累加器）是硬约束，决定时钟周期的最终因素。 ^[raw/articles/ai-chip-architecture-first-principles.md]

有趣的是：高时钟频率（如5-6GHz）意味着几乎所有面积被同步寄存器吃掉，吞吐量反而下降。吞吐量 = 每周期能做的计算 × 时钟频率。和推理批次大小同理：小批次低延迟，但总吞吐量反而下降。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## FPGA vs ASIC：什么时候用哪个

两者成本结构完全不同： ^[raw/articles/ai-chip-architecture-first-principles.md]
- **FPGA**：第一片花1万美元；工作负载经常变化的场景（如高频交易）；同等逻辑消耗面积是ASIC的10倍，能效差一个数量级
- **ASIC**：第一片花3000万美元（流片费用）；大规模量产时单片成本远低于FPGA；实现同样功能比FPGA面积小10倍

FPGA的「编程」本质是配置多路选择器的控制位。查找表（LUT）本质上是一个4输入1输出的真值表，存储16个配置比特——想要与门就写入与门的真值表。一个4输入LUT用mux实现需要约32个门，而ASIC直接做一个四路与门只需3个与门。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## Cache vs 暂存器：确定性延迟的来源

**CPU非确定性的最大来源是缓存。** 缓存比DDR快两个数量级，没有缓存几乎所有程序都会慢100倍。但数据是否命中缓存取决于CPU环境（其他程序在干嘛、最近跑过什么、缓存替换算法的随机性）——这是CPU延迟不确定的根本原因。 ^[raw/articles/ai-chip-architecture-first-principles.md]

**AI芯片换了一种设计哲学：暂存器（scratchpad）。** 给软件两种不同指令：一类访问片上暂存器，一类访问片外HBM。把「数据在哪」的决定权从硬件交还给软件。这也是Groq和TPU可以宣称确定性延迟的原因——它们用scratchpad而不是cache。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## CPU、GPU、TPU：核心大小的差异

| | 核大小 | 缓存 | 分支预测 | 典型核心数 |
|---|---|---|---|---|
| **CPU** | 大 | 有（快100倍） | 有（提前5周期预测） | 几十 |
| **GPU (SM)** | 小 | 无（用scratchpad） | 无（并行同构） | 几千 |
| **FPGA (LUT)** | 极小 | 无 | 无 | 几百万LUT |

GPU核（SM）相对CPU核更小的原因：去掉复杂缓存和分支预测——这些在并行同构的GPU工作负载中不是必需的。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## 大脑 vs 芯片

三个关键区别： ^[raw/articles/ai-chip-architecture-first-principles.md]
1. **稀疏性**：芯片可以做结构化稀疏（整列剪枝），大脑是非结构化稀疏——任何神经元可以连接任何其他神经元 ^[raw/articles/ai-chip-architecture-first-principles.md]
2. **时钟频率**：大脑更慢以节能；但频率降低并不会获得线性能效提升——芯片大部分能耗来自比特翻转，跑得慢只是翻转次数少，不等于高效 ^[raw/articles/ai-chip-architecture-first-principles.md]
3. **批次大小**：GPU跑推理批次是1000，大脑只有一个你 ^[raw/articles/ai-chip-architecture-first-principles.md]

## GPU就是一堆小TPU

从顶层架构看，**GPU是一堆小的TPU平铺在整个芯片上**： ^[raw/articles/ai-chip-architecture-first-principles.md]
- GPU由很多近乎相同的SM（流式多处理器）组成，排列成规则网格，中间是L2缓存
- TPU只有几个粗粒度的矩阵单元（MXU），中间夹着一个向量单元
- 把TPU缩小——矩阵单元变小，向量单元变小——你就得到了一个SM。SM里的Tensor Core本质上就是一个小的脉动阵列

两种设计取舍： ^[raw/articles/ai-chip-architecture-first-principles.md]
- **TPU粗粒度设计**：单个脉动阵列更大，更好摊销寄存器堆成本；但向量单元和矩阵单元之间的数据传输只能走两条边界的路径
- **GPU**：有16条这样的路径，数据搬运灵活性更高

MatX正在尝试：既有GPU那样被SRAM包围的小型脉动阵列的灵活性，又要丢掉SM里为支持CUDA架构而存在的大量占用面积的东西。可拆分脉动阵列（本质上是一个大阵列，但可以按需变成多个小阵列）是核心方向。 ^[raw/articles/ai-chip-architecture-first-principles.md]

## 深度分析

### 乘法器面积与位宽的平方关系：从Dadda压缩器到FP4

4比特乘4比特的乘法需要16个与门产生部分积，真正的代价在求和阶段。Dadda乘法器把需要加的比特排成网格，用全加器（3→2压缩器）反复压缩——输入三个单比特，输出两个比特，因为0+1+1=2的二进制表示是10 ^[raw/articles/ai-chip-architecture-first-principles.md]。

全加器数量公式：p×q个（输入p×q + (p+q)个比特，输出p+q个比特，每用一个全加器消灭一个比特）。这就是为什么**乘法器面积和比特位宽是平方关系**——精度减半，理论上算力增加4倍。NVIDIA在B300及之后已承认FP4性能是FP8的3倍（理论4倍），印证了这一规律 ^[raw/articles/ai-chip-architecture-first-principles.md]。

### 流水线寄存器的双向代价

把逻辑云劈成两半中间插寄存器，时钟频率能翻倍——但代价是多了存储面积。有反馈回路的情况（如累加器）更是硬约束：在加法器中间插寄存器，连续求和就变成两个独立的求和（偶数项的和、奇数项的和），这是决定时钟周期的最终因素 ^[raw/articles/ai-chip-architecture-first-principles.md]。

高时钟频率的代价：如果用寄存器配与门跑到五六GHz以上，几乎所有面积被同步寄存器吃掉，吞吐量反而下降。**吞吐量 = 每周期能做的计算 × 时钟频率**——和小批次推理低延迟但总吞吐量下降是同一道理 ^[raw/articles/ai-chip-architecture-first-principles.md]。

### FPGA「编程」的本质：配置多路选择器

FPGA的「编程」本质是配置mux控制位。LUT是一个4输入1输出的真值表，存储16个配置比特——想要与门就写入与门的真值表。一个4输入LUT用mux实现需要约32个门（16个与门+16个或门），而ASIC直接做四路与门只需3个与门。**同等逻辑下FPGA面积是ASIC的10倍，能效差一个数量级** ^[raw/articles/ai-chip-architecture-first-principles.md]。

成本对比：第一片FPGA花1万美元，第一片ASIC花3000万美元（流片费用）。FPGA适合工作负载经常变化的场景（如高频交易），大规模量产时ASIC单片成本远低于FPGA ^[raw/articles/ai-chip-architecture-first-principles.md]。

### 大脑 vs 芯片：稀疏性、频率与批次

三个关键差异： ^[raw/articles/ai-chip-architecture-first-principles.md]

1. **稀疏性**：芯片做结构化稀疏（整列剪枝），大脑是非结构化稀疏——任何神经元可以连接任何其他神经元，没有对齐的列结构可以利用 ^[raw/articles/ai-chip-architecture-first-principles.md]

2. **时钟频率**：大脑慢以节能；但芯片降频不会线性能效提升——芯片大部分能耗来自比特翻转，降频只是翻转次数少，不等于高效。芯片会快速完成计算后长时间闲置，闲置期间不耗什么电 ^[raw/articles/ai-chip-architecture-first-principles.md]

3. **批次大小**：GPU推理批次1000，大脑只有一个你——这是最根本的架构差异 ^[raw/articles/ai-chip-architecture-first-principles.md]

### TPU粗粒度 vs GPU细粒度：数据搬运路径的取舍

TPU粗粒度设计：单个脉动阵列更大，更好摊销寄存器堆成本；但向量单元和矩阵单元之间的数据传输只能走两条边界的路径。GPU有16条这样的路径，数据搬运灵活性更高 ^[raw/articles/ai-chip-architecture-first-principles.md]。

MatX尝试的方向：既有GPU那样被SRAM包围的小型脉动阵列的灵活性，又要丢掉SM里为支持CUDA架构而存在的大量占用面积的东西。可拆分脉动阵列（本质上是一个大阵列，但可以按需变成多个小阵列）是核心方向 ^[raw/articles/ai-chip-architecture-first-principles.md]。

### 从与门到数据中心：同一个主题

从逻辑门到数据中心，芯片设计的每一个层级都在做同一件事——**最大化计算相对于通信的比例**。数据搬运税在每个层级都存在：逻辑门的mux面积、寄存器堆的读取电路、脉动阵列的权重复用、乃至数据中心机房间的网络带宽。这个主题贯穿全栈，是理解所有硬件优化的核心线索 ^[raw/articles/ai-chip-architecture-first-principles.md]。

## 实践启示

1. **数据搬运是最贵的**：无论是芯片设计还是Agent系统，最大化计算相对于通信的比例是贯穿全栈的核心主题 ^[raw/articles/ai-chip-architecture-first-principles.md]
2. **精度与算力的平方关系**：精度减半算力理论上增4倍；FP4是下一个NVIDIA承认的方向 ^[raw/articles/ai-chip-architecture-first-principles.md]
3. **确定性延迟来自软件显式管理**：scratchpad > cache（对于AI推理工作负载） ^[raw/articles/ai-chip-architecture-first-principles.md]
4. **时钟频率与吞吐量的矛盾**：不要盲目追求高时钟频率，小批次低延迟往往牺牲总吞吐量 ^[raw/articles/ai-chip-architecture-first-principles.md]
5. **硬件专业化趋势**：训练/推理分离的硬件架构（Google TPU 8t/8i）正在成为行业共识 ^[raw/articles/ai-chip-architecture-first-principles.md]
## 相关实体
- [[entities/the-inference-shift]]
- [[entities/triton-l2缓存命中优化矩阵乘法fp16int8详解及性能测试]]
- [[entities/google-io-2026-agentic-gemini-era]]
- [[entities/eks-gpu-operator-custom-driver-cuda-workload]]
- [[entities/modal-truly-serverless-gpus]]
- [[moc/nvidia-gpu-acceleration|MOC]]
