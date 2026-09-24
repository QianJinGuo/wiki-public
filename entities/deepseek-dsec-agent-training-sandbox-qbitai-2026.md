---

title: "DeepSeek DSec Agent训练基础设施论文"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: ['agent', 'post-training', 'deepseek', 'sandbox']
sources: [raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026]
confidence: 0.7
vxc: 56
score: v=8/c=7/stars=4
provenance_state: extracted
---

# DeepSeek DSec Agent训练基础设施论文



# DeepSeek新论文公开Agent训练！梁文锋署名

关注前沿科技 2026-09-23 11:09 北京

每秒能产生5000+个沙盒

##### 克雷西 发自 凹非寺  
量子位 | 公众号 QbitAI

大模型训练拼的是算力，Agent训练拼的是环境。

环境怎么造？梁文锋署名的DeepSeek最新论文，把技术细节公开了。

DeepSeek做的这个系统叫DSec（DeepSeek Elastic Compute），干的事情就是给Agent训练批量制造沙盒。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

它每秒能产生5000+个沙盒，一天能达到300万个，峰值同时运行38万个。

支撑这个规模的单集群也非常庞大，大约有160个节点、3万核CPU和250TB内存。

为啥训个Agent会这么费劲？

因为大模型训练的环境就是GPU集群，喂数据算梯度，但Agent完全不同。

它得在沙盒里写代码、跑编译、开浏览器，甚至装操作系统……每执行一步都改变环境状态，随时可能把环境搞崩。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

所以每轮训练都得给它一个全新的、干净的沙盒，而且随用随抛、训完就扔。

所以，问题兜兜转转，还是回到了基础设施——

这些基础设施需要在每秒5000个的速度下，给每个沙盒装好一整套操作系统和工具链。

同时，还不能让几十万个并发沙盒把集群的内存和CPU挤爆。

具体怎么办，论文把这整套工程的全貌摊开了。

## Agent训练需要「…

## 核心内容



# DeepSeek新论文公开Agent训练！梁文锋署名

关注前沿科技 2026-09-23 11:09 北京

每秒能产生5000+个沙盒

##### 克雷西 发自 凹非寺  
量子位 | 公众号 QbitAI

大模型训练拼的是算力，Agent训练拼的是环境。

环境怎么造？梁文锋署名的DeepSeek最新论文，把技术细节公开了。

DeepSeek做的这个系统叫DSec（DeepSeek Elastic Compute），干的事情就是给Agent训练批量制造沙盒。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

它每秒能产生5000+个沙盒，一天能达到300万个，峰值同时运行38万个。

支撑这个规模的单集群也非常庞大，大约有160个节点、3万核CPU和250TB内存。

为啥训个Agent会这么费劲？

因为大模型训练的环境就是GPU集群，喂数据算梯度，但Agent完全不同。

它得在沙盒里写代码、跑编译、开浏览器，甚至装操作系统……每执行一步都改变环境状态，随时可能把环境搞崩。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

所以每轮训练都得给它一个全新的、干净的沙盒，而且随用随抛、训完就扔。

所以，问题兜兜转转，还是回到了基础设施——

这些基础设施需要在每秒5000个的速度下，给每个沙盒装好一整套操作系统和工具链。

同时，还不能让几十万个并发沙盒把集群的内存和CPU挤爆。

具体怎么办，论文把这整套工程的全貌摊开了。

## Agent训练需要「一个世界」

DSec要解决的第一个核心问题是，不同类型的Agent任务对沙盒环境的要求差异极大，而且这些环境必须在同一个平台上统一调度。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

一个刷OJ题的Agent，只需要一个无状态的函数调用环境，跑完拿到输出就行，连文件系统都不需要持久化。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

但一个做SWE-bench的Agent，就需要完整的Linux用户态，得在里面装依赖、改代码、跑pytest，任务做到一半还可能要往环境里加新包。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

到了安全攻防和computer-use场景，容器级别的隔离就不够了，Agent要操作浏览器甚至桌面，一个有漏洞的Agent可能顺手把宿主机搞挂，必须上虚拟机。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

最极端的情况是训练操作商业软件的Agent，它需要一个完整的Windows或macOS，带图形界面、带驱动，跟真实电脑几乎没区别。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

DSec为这四类场景分别准备了四种后端，FnCall处理无状态函数调用，Container跑Docker容器，MicroVM用Firecracker做轻量级虚拟机，Full VM用QEMU跑完整操作系统。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

四种后端的隔离强度和资源开销逐级递增，但训练框架那边看到的是统一的Python SDK libdsec。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

不管底层是容器还是虚拟机，都采用相同的接口，创建沙盒、执行命令、拿结果，各个步骤的调用方式完全相同。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

要让四种后端在同一套集群上跑起来，平台的调度层也得跟上。

DSec把整个链路拆成了六层。

这条链路从训练框架的一个创建请求出发，先经过IAM认证鉴权，进入API Server，再由调度引擎（Placement Engine）根据资源余量从集群中选出一台目标节点，节点上的Edge组件负责实际拉起对应类型的沙盒。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

沙盒的网络出口和包管理镜像由Aether统一代理，Agent在里面执行的每条命令和产生的每行输出，都通过一个叫Chronus的沙盒内通信组件中转回训练框架，让框架知道Agent做到了哪一步、该给什么反馈。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

靠资源超分和高密度部署，单个节点可以同时承载3200个容器或800个MicroVM。

## 每天300万个沙盒怎么带动？

然而，DSec在规模上最狠的挑战还不是调度，是环境的构建。

每个沙盒启动时，都需要一整套操作系统镜像加工具链，相当于每秒给5000台「电脑」装系统。

传统Docker的思路是把基础镜像、工作区和工具包打成一个完整镜像。

这个方案在小规模下没问题，但DSec的容器后端累计使用了11266个基础镜像和102171个工作区，67.8%的沙盒需要在基础镜像之上叠加至少一层工作区或工具包。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

在这种多样性下，一旦某个工具包更新，所有包含它的组合镜像全部要重新构建，成本是O(m·N)。

DSec的做法是把环境拆成基础镜像、工作区、工具包三层独立的EROFS只读镜像，各自独立版本化，通过overlayfs在沙盒启动时按需组合。更新工具包只碰工具包那一层，成本降到O(m)+O(k)。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

镜像造好之后，怎么送到节点上同样关键。

直觉上应该提前把镜像拉到本地缓存好，但论文统计了真实的运行时数据：

  * Python容器镜像6.0GB，Agent实际只读取了其中6.0%的数据；

  * Java镜像12.1GB，只有9.2%被访问；

  * C++镜像4.9GB，只有8.7%被访问。




也就是说，绝大部分镜像内容，Agent从头到尾碰都没碰过。

所以DSec选择按需加载，其镜像以EROFS格式存储在3FS（Fire-Flyer分布式文件系统）上，元数据预取到本地，数据块只在沙盒真正读取时才从3FS拉过来。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

DeepSeek团队实测，8192个容器的突发部署，按需加载只要35分钟就能完成，而Docker冷拉取要60分钟以上。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

另外，按需加载的磁盘写入量，也比Docker冷拉少了一大半，从约1600GB降到约700GB。

环境建好之后，几十万个沙盒同时跑起来又面临资源争抢。

内存方面，MicroVM通过虚拟块设备读取镜像数据时，同一份数据会在宿主机和虚拟机的页缓存里各存一份，导致需求倍增。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

DSec用virtio-pmem配合DAX让虚拟机跳过自己的页缓存，直接映射到宿主机物理内存，多个虚拟机共享同一份映射，峰值内存占用砍掉40.2%。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

对virtio-pmem不适用的可写磁盘，DSec用DAMON定期扫描冷内存页并主动归还宿主机，配合virtio-balloon的free-page reporting再将需求砍掉21.2%。 ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]

CPU方面，DSec把沙盒分成延迟敏感型和尽力而为型两类，…

→ [[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026|原文存档]] ^[raw/articles/deepseek-dsec-agent-training-sandbox-qbitai-2026.md]