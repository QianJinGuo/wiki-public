---
title: "MCP-based Interactive PDF Text Extraction from Amazon S3"
created: 2026-07-01
updated: 2026-10-23
type: entity
tags: [mcp, aws, s3, pdf, document-processing, ai-agent]
sources: [raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3]
confidence: 0.8
provenance_state: extracted
review_value: 7
review_confidence: 8
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# MCP-based Interactive PDF Text Extraction from Amazon S3

AWS 推出的基于 Model Context Protocol (MCP) 的交互式 PDF 文本提取方案，让 AI Agent 能够实时从 Amazon S3 存储的 PDF 文档中提取文本内容，无需等待批量处理管道。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:1-20]

## 摘要

该方案通过 MCP 服务器提供程序化文档访问能力，使 AI 助手能够直接查询存储在 S3 中的 PDF 文件。相比 Amazon Textract，在概念验证场景下成本降低约 90%，同时保持实时交互能力。适用于合规审查、法律咨询、财务分析等需要即时文档访问的场景。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:1-50]

## 核心能力

### 实时文本提取

该方案通过 MCP 服务器提供程序化文档访问能力，使 AI 助手能够直接查询存储在 S3 中的 PDF 文件：^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md]


- **即时响应**：典型 50 页文本 PDF 在几秒内完成提取^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:288-290]
- **自然语言查询**：用户可用自然语言提问，获取文档中的相关段落^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:24-26]
- **线性扩展**：处理时间与文档大小成线性关系，10 页文档比 50 页快约 5 倍^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:289-290]
- **内存友好**：100 页以下文本 PDF 内存占用在典型开发机限制内^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:290-291]

### 成本优化

| 方案 | 月处理量 | 成本构成 | 总成本 |
|------|----------|----------|--------|
| **Amazon Textract** | 10,000 页 | Textract $15 + S3 $2 + Lambda $1 + LLM $5-10 | $23-28 |
| **MCP Server** | 10,000 页 | S3 $2 + 数据传输 $0.50 | $2.50 |

^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:42-56]

成本对比说明：两方案功能集不同，不应作为直接价格竞争。涉及扫描文档、表单、表格、复杂布局或生产 SLA 时，Amazon Textract 是更合适的选择。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:46-47]

### 最小化基础设施

无需托管处理服务，直接连接 AI 助手与源文档。架构仅包含四个组件：^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md]


1. **命令行界面**：用户与系统交互的入口^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:40-40]
2. **MCP 通信层**：标准化的外部数据源访问协议^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:40-40]
3. **自定义 MCP 服务器**：专门处理 PDF 文档的服务器^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:40-40]
4. **Amazon S3**：文档存储，通过 AWS IAM 进行安全控制^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:40-40]

## 技术实现

### 工作流程

```
用户 → CLI → MCP Client → MCP Server → S3 (下载 PDF)
                                    ↓
                              PDF Parser (PyPDF2)
                                    ↓
                         返回提取的文本 → 显示给用户
```

^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:60-63]

### MCP 服务器核心代码

```python
from mcp.server import Server
from mcp.types import Tool, TextContent
import boto3
from PyPDF2 import PdfReader
import tempfile
import os

server = Server("s3-pdf-extractor")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="extract_s3_pdf_text",
            description="Extract text content from a PDF stored in Amazon S3",
            inputSchema={
                "type": "object",
                "properties": {
                    "bucket": {"type": "string", "description": "S3 bucket name"},
                    "key": {"type": "string", "description": "S3 object key"}
                },
                "required": ["bucket", "key"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "extract_s3_pdf_text":
        bucket = arguments["bucket"]
        key = arguments["key"]

        s3_client = boto3.client('s3')
        with tempfile.NamedTemporaryFile(delete=False, suffix='.pdf') as tmp_file:
            s3_client.download_file(bucket, key, tmp_file.name)
            tmp_path = tmp_file.name

        reader = PdfReader(tmp_path)
        text = ""
        for page in reader.pages:
            text += page.extract_text() + "\n"

        os.unlink(tmp_path)
        return [TextContent(type="text", text=text)]
```

^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:154-217]

### 部署步骤

**环境准备**：
- Python 3.10+
- AWS CLI 配置有效凭证^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:72-76]
- Kiro CLI 安装^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:75-75]
- 依赖安装：`pip install mcp boto3 PyPDF2`^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:76-76]

**Kiro CLI 配置** (`~/.kiro/settings/tools/mcp.json`)：^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md]

```json
{
    "mcpServers": {
        "s3-pdf-extractor": {
            "command": "python",
            "args": ["/path/to/s3_pdf_extractor.py"]
        }
    }
}
```

^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:219-247]

**验证部署**：
```bash
extract text from s3://your-bucket-name/sample.pdf
```

^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:268-271]

## 适用场景对比

### 推荐使用 MCP 方案的场景

- **文本型 PDF**：PDF 中已编码文本，无需 OCR^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:36-37]
- **交互式工作流**：实时查询而非批处理^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:36-37]
- **开发和 PoC 环境**：成本敏感场景^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:36-37]
- **现有 AWS 工作流集成**：利用现有 IAM 和 S3 基础设施^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:36-37]

### 推荐使用 Amazon Textract 的场景

- **扫描文档**：需要 OCR 的光栅化图像^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:34-35]
- **手写识别**： Textract 原生支持^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:34-35]
- **表单和表格提取**：结构化数据提取^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:34-35]
- **复杂布局分析**：多栏、复杂排版^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:34-35]
- **生产级 SLA**：企业级服务等级协议^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:34-35]

### 混合工作流

对于同时包含扫描页和文本页的 PDF 仓库，可设计自动路由：^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md]

- 先尝试 MCP 方案提取文本
- 遇到失败或图像页时自动转交 Textract^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:328-328]

## 安全考虑

该方案从设计之初就集成安全性：

1. **IAM 集成**：使用现有 AWS 凭证，无需单独 API 密钥^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:274-276]
2. **最小权限**：仅授予特定 S3 桶的读权限^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:277-278]
3. **临时存储**：处理完成后自动删除下载文件^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:278-279]
4. **无数据持久化**：按需提取，不存储结果^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:279-280]
5. **审计追踪**：AWS CloudTrail 记录所有 S3 访问请求^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:280-281]

## 局限性与边界

**明确限制**：

- **仅文本型 PDF**：扫描图像或照片无法读取，需使用 Amazon Textract^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:296-297]
- **无 OCR 能力**：只能读取 PDF 格式中嵌入的文本，无法解释图像像素^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:297-298]
- **有限布局理解**：直接文本提取，不重建表格、栏或复杂页面布局^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:298-299]
- **无表单处理**：不提取可填写表单字段或结构化数据^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:299-300]

## 真实用例

### 法律服务公司

中型律师事务所用于合同审查。律师过去需在客户电话中花费 15-20 分钟搜索 PDF 合同中的特定赔偿条款，现在通过 Kiro CLI 输入问题，几秒内获得相关段落。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:307-310]

### 金融服务合规

地区银行用于监管检查。合规官员需要在审计期间快速定位特定政策语言，过去需手动在数十个 PDF 文件中添加书签，现在通过 MCP 服务器连接 S3 文档库，实时提取 examiner 询问的确切段落。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:311-314]

### 企业战略团队

企业领导团队在季度战略会议中使用。当董事会成员询问上季度财报中的特定指标时，团队可现场查询 PDF，无需翻阅纸质副本。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:315-317]

## 扩展与增强选项

该方案是起点，可根据需求扩展：

- **添加缓存**：使用 Amazon DynamoDB 缓存频繁访问的文档^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:325-325]
- **批处理**：使用 Amazon SQS 实现批量操作^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:326-326]
- **向量搜索**：集成 Amazon OpenSearch Service 实现语义文档发现^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:327-327]
- **混合工作流**：自动将复杂文档路由到 Amazon Textract^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:328-328]
- **监控**：使用 Amazon CloudWatch 跟踪使用模式和错误率^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:329-329]

## 深度分析

### 为什么是协议而不是胶水：MCP 作为工具契约

四组件架构里最易被低估的是 MCP 层：它解决的不是"怎么调 S3"，而是工具契约放在哪里。定制 Lambda 胶水的路线上，每新增一个文档操作都要在 handler、API 定义、prompt 里的工具说明三处同步修改，一处漂移模型便去调不存在的参数。MCP 把 `list_tools` 的 name / description / inputSchema 变成运行时产物：schema 即契约，服务端只在 `call_tool` 上分支，新增能力退化成"一条 Tool 描述 + 一个 if"。参见 [[concepts/model-context-protocol-mcp|MCP]]。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:40-40]

代价是绑定部署形态。`server.run()` 加 `"command": "python"` 的配置说明它是本地 stdio 长驻进程：没有 Lambda 式冷启动（PyPDF2 与 boto3 的 import 只在启动付一次），但并发实质为 1：单进程、阻塞式 `download_file`、`PdfReader` 全量载入，两个请求只能串行。

### 无状态调用：分页与分块在资源模型里没有位置

`inputSchema` 只有 `bucket` 和 `key`，没有 offset、page 或游标，这决定了它是"整文档一次吐完"而非可寻址资源。文章开头举的恰是"200 页政策文件里找一条条款"——全量文本压进单次响应，token 与体积随页数线性膨胀，模型却往往只需两页。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:154-217]

资源模型本可表达 `s3://bucket/key#page=37` 这类子资源，配合游标做成"先检索、后取页"；当前没走这条路，chunking 便全部下沉到客户端：让 LLM 在大段文本里自己定位（烧 context），或外挂向量检索——文章把 OpenSearch 列为扩展项，恰好承认了这个缺口。被丢掉的还有页边界：`text += page.extract_text()` 返回无标记的连续文本，而法务、合规的核心价值正是引用到具体条款与具体页。最小修补是每页前插 `[p.N]`，成本近乎为零，却把可回溯性还给下游。

### 预签名 URL 还是代理转发：凭证位置决定爆炸半径

当前实现是代理转发：凭证始终留在 server 进程内，agent 只看到两个字符串。优势是凭证不出进程；代价是数据平面穿过 server（临时文件落盘、`finally` 里 unlink），且该进程须有权读所有传入的 bucket。关键是 bucket 在 schema 里是自由字符串——不做白名单，per-tool 最小权限最终只能落在 IAM policy 的 resource ARN 上，那是方案里唯一真正的闸门。

presigned URL 正好反过来：server 不碰字节，egress 转移到客户端、S3 直连；但 URL 本身是 bearer token，进入模型上下文、trace 或日志就等于把对象访问权交给所有读得到它的人。TTL 是唯一止损阀门，而它必须长到覆盖多轮推理，两者天然冲突。文章选代理转发，判断很清楚：文档是敏感资产，凭证关在进程内比省一段 egress 更值钱。

### 文本层与 OCR 的边界：保真度断在哪里

`extract_text` 读的是 PDF 内容流里的文本对象，判定边界很干脆：有文本层就能读，没有就一个字都读不到。真正的陷阱不是纯扫描件（一眼可辨、直接路由 Textract），而是混合 PDF：200 页合同里夹了三页扫描附件。该页返回空串却不抛异常，代码走到 `logger.info("Successfully extracted ...")` 正常返回。静默成功是最坏的失败模式——下游 LLM 会把"提取为空"读成"文档中没有该条款"。

保真度还有三处断点：多栏按内容流而非视觉顺序拼接，两栏交错读串；表格被拍平成单元格文本，行列与表头一起丢失；AcroForm 表单域不在视野内——这正是 forms / tables / complex layouts 被划给 Textract 的分工边界。近乎免费的路由启发式：循环本就逐页，顺手统计每页字符数，低于阈值（如 50 字符）即标记需 OCR，只送这几页去 Textract——约 $0.0015/页，混合文档通常只有个位数比例需要，总成本仍远低于全量 OCR。

### 运行包络：成本结构、并发上限与审计

成本表里 $2.50 = S3 存储 $2 + 数据传输 $0.50，边际成本几乎全是 egress，无按页计费调用——这是相对 Textract $23-28 的全部价差来源，也意味着账单随"读了多少 GB"而非"处理多少页"增长。容量规划该估平均文档大小乘查询次数；重复查询时 DynamoDB 缓存省的是出口流量，不是算力。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:44-56]

并发受三条物理约束：单进程串行、临时文件占磁盘、`PdfReader` 全量载入内存。文章的经验值——100 页以内落在开发机内存限制内、耗时随页数线性（10 页约比 50 页快 5 倍）——意味着延迟由页数决定，吞吐由"能否多开进程"决定；多开进程会同时放大临时文件竞争与内存占用。这才是它被定位在 PoC 而非生产 SLA 的技术原因，与功能缺失无关。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:284-290]

安全五条里真正承重的是第一条与第五条：复用 IAM 意味着没有第二套密钥可泄露；CloudTrail 记录 GetObject，每次提取都留下"谁、何时、读了哪个 object"，这是审计型用例能接受该方案的前提。租户隔离不在方案内：一个进程、一套凭证、所有 bucket 平权。补法是每租户一个 server 实例加一份只含该租户前缀的 IAM policy，让隔离落在进程与 ARN 上，而非在 `call_tool` 里写 if。延伸阅读：[[concepts/agent-security-architecture|Agent 安全架构]]、[[entities/mcp-2026-07-28-stateless-extensions-ema|MCP 无状态扩展]]。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:272-280]

## 实践启示

### 选型决策树

```
PDF 类型？
├── 扫描/图像/手写 → Amazon Textract
└── 文本编码 PDF → 用途？
    ├── 批处理/生产 SLA → Amazon Textract
    └── 交互式/实时查询 → MCP 方案
```

### 关键要点

1. **成本与功能权衡**：MCP 方案成本极低但功能有限，Textract 功能全面但成本更高^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:42-56]
2. **协议标准化价值**：MCP 作为开放标准，使文档访问与具体 LLM 或 Agent 框架解耦^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:39-40]
3. **最小可行基础设施**：对于文本提取这类相对简单的任务，无需重量级托管服务^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:365-368]
4. **渐进式增强**：从 MCP 方案开始，随需求增长逐步添加缓存、批处理、向量搜索等能力^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:319-329]

→ [[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

