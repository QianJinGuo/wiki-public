---

title: "AliSQL 向量技术解析（一）：存储格式与算法实现"
type: entity
created: 2026-07-04
updated: 2026-09-19
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/alisql-向量技术解析一存储格式与算法实现
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# AliSQL 向量技术解析（一）：存储格式与算法实现

**来源**: 阿里技术 ｜ **发布日期**: 2025-11-19 ｜ **原文链接**: https://mp.weixin.qq.com/s/4fLgnwhIuJe76dX89Mv16A ｜ 基于 AliSQL 8.0 20251031 版本 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:16-18,36]

## 摘要

AliSQL 在 MySQL 8.0 上原生扩展了企业级向量能力：新增 `VECTOR` 列类型，最高支持 16,383 维向量的存储与计算，内置 COSINE 与 EUCLIDEAN 运算函数，并支持对全维度向量列建立基于 HNSW（Hierarchical Navigable Small World）算法的向量索引，全部通过标准 SQL 暴露 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42]。它填的是 MySQL 生态的长期空白——PostgreSQL 有 pgvector，MySQL 9.0 虽引入 `VECTOR` 类型但距离计算只在 HeatWave 可用、没有通用索引，企业此前只能独立部署向量库或迁数据 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:32]。对存量实例而言，这意味着不必搬数据，就能在同一 OLTP 引擎里把向量召回与业务逻辑拼进一条 SQL ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:34]。

## 核心要点

- **维度上限**：最高 16,383 维的存储与计算，索引只建立在全维度向量列上 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42]
- **距离度量**：函数侧含 COSINE 与 EUCLIDEAN；索引侧示例只出现 `distance=cosine` ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42]
- **索引算法**：优先支持 HNSW，原文称其为最流行的 ANN 算法之一，已在评测与工程实现中广泛验证 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:62]
- **DDL**：列内声明 `VECTOR INDEX \`vi\`(\`vec\`) m=6 distance=cosine`；也可后建 `CREATE VECTOR INDEX`，省略参数则走默认值 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:46,56]
- **DML/查询**：`VEC_FROMTEXT` 写入向量，`VEC_DISTANCE(...)` 配合 `ORDER BY` 做近邻查询 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:47-48]
- **存储实现**：逻辑上一张完整 HNSW 图，物理上是一张对用户不可见的辅助表，每行代表一个节点；内存另有 Nodes Cache 加速 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:66-68]
- **事务性**：辅助表与主表的结构变更封装于同一事务，靠事务系统与 DDL log 保障 DDL 原子性与 Crash Safe ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:192]
- **兼容性**：基于 MySQL 8.0 分支扩展，面向存量 AliSQL 实例开箱即用，与 vanilla MySQL 9.0 的 VECTOR 能力不等价 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:32,198]

## 深度分析

### 存储格式：把 HNSW 图摊成一张 InnoDB 辅助表

实现 HNSW 的关键问题是"图存在哪"。AliSQL 的答案是：不塞进引擎私有结构，而是设计一张辅助表，**每行对应图中的一个节点**，节点包含从第 0 层到它自身最高层的所有映射点 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:138]。四个关键列各有分工：

- `gref`（graph ref）是辅助表主键，供邻居搜索使用，直接复用 InnoDB 系统列 `ROW_ID` ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:144]
- `layer` 上建 KEY，用于快速定位整张图的入口节点，即 layer 最大的节点 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:148]
- `tref`（table ref）是原表主键，用于搜索完成后**回表**，采用 server 层存储格式——主表主键为 1 的 int 列在引擎层是 `80 00 00 01`，辅助表按小端序存成 `01 00 00 00` 的 BINARY ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:152]
- `neighbors` 逐层记录邻居：先存 1 字节邻居数量，再存每个邻居的 `gref`，算法流转时在同层邻居间导航 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:160]

向量本身是有损压缩的：辅助表上的向量以半精度 int16 存储，首 4 字节放 float32 缩放因子 `scale`，线性映射 `int16 = round(float32 / scale)`，`scale` 为向量元素最大绝对值与 32767 之比 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:156,162]。原文以 `[0.6, 0.7]` 走过一遍：`scale = 2.13629555e-05` → 量化得 `[28086, 32767]` → 最终字节 `[2860 C2 37]`（scale）`[B6 6D] [FF 7F]`（dims，小端序）^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:164-166]。

这套设计的两条腿是"持久化在表、加速在缓存"：原文说不断从辅助表加载节点信息即可运行 HNSW 算法，同时在架构上引入内存 Nodes Cache，避免每次图遍历都走磁盘 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:68,168]。

### HNSW 参数：`m` 与 `distance` 买到了什么

HNSW 由论文《Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs》提出，设计可概括为两点：**分层跳表**——第 0 层含全部点，越高层越是下层的缩略图，顶层快速跳转、底层精确搜索；**连接近邻**——每层按向量距离连成邻近图，每点记录最近的几个点作为邻居 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:72-76]。原文复述的三个算法决定了 `m` 的实际作用：

- **插入（Algorithm 1）**：确定新节点 q 的最大层级 l；在 L 到 l+1 层用 `ef=1` 贪心搜索得到下层起点；在 l 到第 0 层用 `ef=M` 做 Search Layer，插入 q 并与邻居**双向连接**，第 0 层特殊用 `ef=2M`；随后对邻居做 **shrink**，连接数超过 M（第 0 层 2M）就移除最远连接以保证图紧凑 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:104-120]
- **单层搜索（Algorithm 2）**：输入目标节点 q、返回结果数 `ef`、候选集 C、结果集 W（初始为空）；循环取 C 中离 q 最近的节点 c，遍历其邻居 e，未被访问且距离更优则同时加入 W 与 C；当 C 中最近节点比 W 中最远节点还远时终止，输出 W 前 `ef` 个 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:82-98]
- **KNN 搜索（Algorithm 5）**：从最高层 L 到第 1 层用 `ef=1` 快速跳转，再在第 0 层搜出 K 个最近邻返回 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:126-134]

于是 `m`（示例 `m=6`）就是"每层最多保留多少邻居"，它同时决定图的连通质量与 `neighbors` 列的宽度：每层要存 1 字节数量加每个邻居一个 `gref`，`m` 越大辅助表与内存中的图越大、写入时 shrink 触发越频繁，换来更密的图、更高召回与更稳延迟 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:120,160]。原文没交代的部分要标明：`m` 与 `distance` 的默认值未给出（只说"不显示指定则默认使用参数值"），查询期也没有 `ef_search` 这类独立旋钮名，只有返回结果数 `ef` 与最终返回的 K ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:56,86]。

### 搜索流水线：代价估计 → 图遍历 → 回表

接入优化器是它能"像普通索引一样被使用"的前提：ANN 查询先经代价估计由优化器选择索引，也可用 `FORCE INDEX` 等 hint 强制指定向量索引 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:64]。查询时通过主表打开辅助表，从 `layer` 最大的入口节点进入，逐层下潜到第 0 层拿到 K 个候选，再用候选节点的 `tref` 回表取原表行——这就是"向量召回"与"业务字段"最终能拼进同一条 SQL 的物理基础 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:152,178]。完整的最小用法：

```sql
-- 创建带有向量索引的表
CREATE TABLE `t1` (
  `id`     INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `animal` VARCHAR(10),
  `vec`    VECTOR(2) NOT NULL,                      -- 新增 vector 类型列
  VECTOR INDEX `vi`(`vec`) m=6 distance=cosine      -- 显式指定 m 和 distance
);

-- 插入数据
INSERT INTO `t1`(`animal`, `vec`) VALUES
  ("Frog", VEC_FROMTEXT("[0.1, 0.2]")),
  ("Dog",  VEC_FROMTEXT("[0.6, 0.7]")),
  ("Cat",  VEC_FROMTEXT("[0.6, 0.6]"));

-- 向量搜索
SELECT `animal`, VEC_DISTANCE(`vec`, VEC_FROMTEXT("[0.1, 0.1]")) AS `distance`
FROM t1 ORDER BY `distance`;
```

原文的结果示例按距离升序为 Cat `0.00000001552204198507212`、Dog `0.0029455257170004634`、Frog `0.05131670194948623`，即查询向量离 `[0.6, 0.6]` 最近 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:47-52]。索引也可后建 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:54-56]：

```sql
ALTER TABLE `t1` MODIFY COLUMN `vec` VECTOR(2) NOT NULL;
CREATE VECTOR INDEX vi ON t1(vec);   -- 不显式指定 m 和 distance 则用默认参数值
```

> 注：原文该示例写作 `CREATE VECTOR INDEX vi ON t1(v);`，按上下文应为 `t1(vec)` ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:56]。

两个边界要留意。其一，原文没有说明检索阶段是用 int16 近似距离还是还原后以 float32 重算，只交代辅助表存 int16 + scale、量化"可以牺牲一定精度，有效降低存储空间并提升搜索效率" ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:156]。其二，示例表仅 3 行，`ORDER BY VEC_DISTANCE(...)` 在这种规模下等价于全量计算，原文并未给出"何时退回全量扫描、何时必须建索引"的阈值 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:52]。

### 为什么放进 OLTP 引擎，边界在哪

驱动力是生态缺口：MySQL 侧没有通用向量索引，企业只能独立部署向量库或迁数据 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:32]。AliSQL 的路径不是外挂向量服务，而是让 HNSW 图成为主表的一部分——辅助表名由主表 table id 拼成 `vidx__00`，主表据此从数据字典（DD）找到其元数据；辅助表的 `TABLE_SHARE` 与 `TABLE` 挂在主表的 `hlindex` 指针下，公共 Nodes Cache 通过 `hlindex_data` 访问，且 `TABLE_SHARE` 不进入 table cache、随主表一起关闭 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:174,182,190]。生命周期绑定主表带来元数据一致性与"数据无需搬运"的迁移成本优势，也让 DDL 落在同一事务里保证原子与 Crash Safe ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:192]。代价与边界同样清楚：

- **维度硬上限 16,383 且只针对全维度列**，超出只能降维或改用专用向量库 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42]
- **量化有损**：int16 + scale 本身带来一层精度误差，与图上近似的误差叠加 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:156]
- **额外开销**：辅助表要存 scale、int16 向量与逐层 `neighbors`，内存还要维持 Nodes Cache ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:68,160]
- **辅助表对用户不可见、无法直接访问**，图状态只能通过主表间接维护 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:172]

索引构建成本与内存占用，原文没有量化；本文也只覆盖存储与算法——可靠性还依赖事务与并发控制，原文明确把 Nodes Cache 优化原理、并发控制与事务隔离留到第二篇 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:200]。

## 实践启示

1. **钉死"维度 + 度量"三处一致**：建表、写入、查询必须同维度同度量；换 Embedding 模型或改维度基本等于重建索引 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42,156]
2. **`m` 从示例值起步再用真实数据压测**：调大会加宽 `neighbors` 列、放大内存图并更频繁触发 shrink，换来更密图与更好召回 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:46,160]
3. **度量按语义选，别照抄示例**：索引侧示例只有 `distance=cosine`，函数侧另有 EUCLIDEAN，切换前先确认索引侧支持该取值 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42,46]
4. **存量大表走后建索引路径**：`ALTER TABLE ... MODIFY COLUMN vec VECTOR(n)` 再 `CREATE VECTOR INDEX`；主表与辅助表结构变更在同一事务且有 DDL log 保障，DDL 原子且 Crash Safe ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:56,192]
5. **混合查询要验证"过滤发生在哪一步"**：`tref` 的语义是"向量搜索完成后回表"，SQL 侧过滤条件的生效位置需要实测，不能假设它参与了候选生成 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:152]
6. **把 16,383 维与量化误差当红线**：超维度上限或无法接受 int16 精度损失时，应评估降维或外挂专用向量库 ^[raw/articles/alisql-向量技术解析一存储格式与算法实现.md:42,156]

## 相关实体

- [[entities/milvus-3-0-search-aggregation-pushdown-shuge-2026|Milvus 3.0 搜索聚合下推]] — 专用向量库一侧的查询下推实现，可对照本文"回表"路径
- [[entities/vector-storage-agent-retrieval-qianwen-aliyun-2026|向量存储支撑 Agent 检索链路]] — 同一体系下 HNSW / DiskANN / IVF 三种 ANN 路线取舍
- [[entities/在-rds-postgresql-中实现-rabitq-量化|在 RDS PostgreSQL 中实现 RaBitQ 量化]] — 传统数据库内做向量量化的另一条路线
- [[entities/chroma-to-qdrant-1m-vector-migration|Chroma 到 Qdrant 百万级向量迁移]] — "数据搬出 OLTP 引擎"的成本对照
- [[concepts/rag-retrieval-augmented-generation|RAG（检索增强生成）]] — 向量检索最典型的上层消费场景

→ [[raw/articles/alisql-向量技术解析一存储格式与算法实现|原文存档]]
