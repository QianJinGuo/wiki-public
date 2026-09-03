---
title: "Wiki Audit Skill"
created: 2026-05-23
updated: 2026-08-01
type: concept
tags: [wiki, maintenance, analysis, skill]
summary: 生成 wiki 真实客观的质量分析报告。使用正确的 regex + shell 交叉验证，逐项透明核查，不估算、不假设。
provenance_state: synthesized
review_value: 8
sources: []
review_confidence: 9
---

## 用途

生成真实、客观、可验证的 wiki 质量分析报告。每次运行都必须：

1. **Shell 优先**：先 grep/wc 确认数量级
2. **Python 复核**：用 Python 做细粒度分析
3. **交叉验证**：两者结果对比，不一致时重跑

## 分析维度

### 1. lint 状态（必须先跑）

```bash
node [canonical wiki 路径已隐藏] [canonical wiki 路径已隐藏] 2>&1 | grep -E "error|warning|tracked page"
```

- 目标：0 errors（warnings 可忽略）
- 如果 lint 报错，先修 lint

### 2. 基础数量（shell 验证）

```bash
cd [canonical wiki 路径已隐藏]
echo "entities: $(ls entities/ | wc -l)"
echo "raw/articles: $(ls raw/articles/ | wc -l)"
echo "concepts: $(ls concepts/ | wc -l)"
grep "^Total pages" index.md
```

- 记录：entities 总数、raw articles 总数、concepts 总数、index.md 报告的总页数
- 对比：lint actual count vs index.md header

### 3. Frontmatter 完整性（Python）

```python
import os, re, glob, yaml

wiki = "[本地路径已隐藏]"
wikilink_pat = re.compile(r'\[' + r'\[([^\]]+)\]' + r'\]')  # 正确 regex: [^\]] 不是 [^|\]]

def get_fm(path):
    m = re.match(r'^---\n(.*?)\n---', open(path).read(), re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else {}

def extract_target(link):
    """Handle bar-separated wikilinks: target|display → target"""
    return link.split('|')[0].strip()

# 3a. 空 tags
empty_tags = [p for p in glob.glob(f"{wiki}/entities/*.md")
              if (get_fm(p).get('tags') or []) in ([], None)]

# 3b. 缺 title
missing_title = [p for p in glob.glob(f"{wiki}/entities/*.md")
                 if not get_fm(p).get('title')]

# 3c. YAML 解析失败
parse_fail = []
for p in glob.glob(f"{wiki}/entities/*.md"):
    try:
        get_fm(p)
    except:
        parse_fail.append(os.path.basename(p))
```

### 4. Wikilink 分析（shell + Python 双验证）

**Shell 计数（数量级确认）：**

```bash
cd [canonical wiki 路径已隐藏]
# entity→entity links: use [entities/ as a stable substring
echo "entity→entity links: $(grep -r '[entities/' entities/ | wc -l)"
echo "entity→raw links: $(grep -r '[raw/' entities/ | wc -l)"
# For link-level count (not line-level), use -oE with proper ERE
echo "entity→entity links (grep -oE): $(grep -rE '\\[\\[entities/[^]]+\\]\\]' entities/ | wc -l)"
```

**Python 详细分析（必须 strip `|` 且去掉 `entities/` 前缀）：**

```python
def extract_target(link):
    t = link.split('|')[0].strip()   # target|display → "target"
    if t.startswith('entities/'):
        t = t.split('/')[-1]         # "entities/slug" → "slug"
    return t

all_names = {}
for subdir in ['entities', 'concepts', 'queries', 'comparisons']:
    for p in glob.glob(f"{wiki}/{subdir}/*.md"):
        all_names[os.path.splitext(os.path.basename(p))[0]] = subdir

in_counts = {n: 0 for n in all_names}
for path in glob.glob(f"{wiki}/entities/*.md"):        # ONLY from entities/, not other dirs
    with open(path) as f:
        content = f.read()
    for link in wikilink_pat.findall(content):
        target = extract_target(link)
        if target in in_counts:
            in_counts[target] += 1

orphans = sum(1 for v in in_counts.values() if v == 0)
hubs = sum(1 for v in in_counts.values() if v >= 3)
isolated = sum(1 for n in all_names if in_counts.get(n,0)==0 and not any wikilink_pat.findall(open(p).read()) for p in [f"{wiki}/entities/{n}.md"] if os.path.exists(f"{wiki}/entities/{n}.md"))
```

### 5. Citation 分析

```python
# CORRECT: escape [ to match literal caret+bracket pattern
cite_pat = re.compile(r'\^\[raw\/articles\/')

for path in entities:
    body = re.sub(r'^---\n.*?\n---\n', '', open(path).read(), count=1, flags=re.DOTALL)
    paras = [p.strip() for p in re.split(r'\n\n+', body)]
    non_trivial = [p for p in paras
                   if p and not p.startswith('#')
                   and not p.startswith('```')
                   and len(p) >= 30]
    entity_has_cite = any(cite_pat.search(p) for p in non_trivial)
```

### 6. Broken links 验证

```bash
cd [canonical wiki 路径已隐藏]
# 列出 entity→entity links 中目标不存在的
grep -rE '\\[\\[entities/([^|\]]+)\\|' entities/ \
  | sed 's/.*\[\[entities\///;s/|.*//' \
  | sort -u > /tmp/targets.txt
comm -23 /tmp/targets.txt <(ls entities/ | sed 's/.md$//' | sort) | head -20
```

### 7. Raw article 完整性

```bash
cd [canonical wiki 路径已隐藏]
wc -l *.md | sort -n | head -10
# <10 行的 raw 可能是抓取不完整
```

## 报告格式

```
=== WIKI AUDIT REPORT ===

## 1. lint 状态
- errors: N
- warnings: N

## 2. 基础数量
- entities: N
- raw/articles: N
- concepts: N
- index.md total: N

## 3. Shell Wikilink 计数
- entity→entity: N条
- entity→raw: N条

## 4. Frontmatter
- 空 tags: N
- 缺 title: N
- YAML 解析失败: N

## 5. Wikilinks (Python, 交叉验证)
- Orphan (0 in-links): N/N (X%)
- Hubs (>=3 in-links): N
- Isolated (0 in + 0 out): N
- Broken links: N

## 6. Citations
- entities 有引用: N/N (X%)
- cited paragraphs: N/N (X%)

## 7. Raw 完整性
- <10行 raw: N 个

## 8. 结论
[根据以上数据给出真实、不夸大的结论]
```

## 关键注意事项

1. **永远先跑 shell 确认数量级**，不要直接相信 Python 结果
2. **wikilink regex 用 `[^\\]]` 而不是 `[^|\\]]`**
   - `[^|\\]]` = "not `|`, not `\`（转义前的反斜杠）, not `]`" — 在 `target|display` 格式中会在 `|` 处停止
   - `[^\\]]` = "not `\`（反斜杠）, not `]`" — 正确匹配到 `]]` 之前的所有内容
   - 简单记法：wikilink 内容里 `|` 和 `/` 都是合法字符，不要在字符类里 exclude 它们
3. **wikilink 目标必须 split('|')[0] 且去除 entities/ 前缀**
   - wikilink 格式：`target|display` → `extract_target()` 返回 `target`
   - 第一步：`link.split('|')[0].strip()` → `"entities/slug"`（去掉 display 部分）
   - 第二步：`if target.startswith('entities/'): target = target.split('/')[-1]` → `"slug"`（去前缀）
   - **错误**：直接用 `"entities/slug"` 去查 all_names，因为 all_names 的 key 是 bare slug（如 `"slug"`）
   - **后果**：导致 in_counts 全为 0，orphan 率虚报 96%，entity→entity links 虚报 28 条
4. **Python wikilink 分析 source 只从 entities/ 目录遍历**
   - all_names 包含 entities + concepts + queries + comparisons（被链接目标池）
   - 但 in_counts 的 source 只遍历 `entities/` 下的文件（因为 wikilinks 主要在 entity 文件里）
5. **citation regex 必须转义 `[`**: `r'\^\[raw\/articles\/'`
   - `[^raw]` 在字符类里是"非 r 非 a 非 w"，不是"非 `r` 字母"，导致 `^` 被误解为行首
6. **Shell grep 测 wikilink 数量时，优先用 `[entities/` 子串**
   - macOS bash 下复杂 bracket escaping 容易写错
   - 直接用 `'[entities/'` 或 `'[raw/'` 匹配实体和原文链接子串
   - 精确链接级计数可用 double-escaped bracket regex，但要避免把示例写成可被 lint 识别的真实 wikilink

## 相关页面

- [[queries/wiki-maintenance-faq|Wiki 维护 FAQ]]
- [[queries/wiki-capability-self-assessment|Wiki 能力自评]]

## 新发现的 corruption 模式（2026-05-25）

### `tags: [...]` 内联数组 + 缩进块序列混用

**检测**:

```python
re.search(r'^tags:\s*\[\s*[^\]]+\]\s*\n\s+- ', fm, re.MULTILINE)  # Pattern 1
re.search(r'^tags:\s*\[\s*\]\s*\n\s+- ', fm, re.MULTILINE)       # Pattern 2
```

**Pattern 1 示例**:

```yaml
tags: [browser-automation, cdp, agent-harness]
  - browser-automation
  - cdp
  - agent-harness
```

**Pattern 2 示例**:

```yaml
tags: []
  - default
  - inference
```

**修复**: 删除 inline `tags: [...]` 行，提取缩进块序列项，转换为规范内联格式 `tags: [item1, item2]`。

**验证**: `yaml.safe_load()` 必须成功。

### EXCESS INFERRED 是 wikilink 型 wiki 的假阳性

**场景**: wiki 使用 `wikilinks` 作为溯源方式，而非 `citations` Zettelkasten citation 格式。

**表现**: wiki-lint 报告大量 EXCESS INFERRED，但实际 metadata errors=0, true orphans=0, broken links=0。

**原因**: lint 的 citation 检测只认 `^[...]` 格式，raw wikilinks 被计入 wikilink 分析（outDegree/inDegree）但不计入 provenance citation 统计，导致大量段落被标记为"无溯源"。

**判断树**:

```
lint 报告 EXCESS INFERRED 大量警告
├── wiki 使用  wikilinks  作为 provenance → 假阳性，忽略
└── wiki 使用  citations  作为 provenance → 真实问题，需补充 citation markers

metadata errors > 0 → 真实问题，先修
true orphans > 0 → 真实问题，先修
broken links > 0 → 真实问题，先修
```

对于 wikilink 型 wiki，在"6. Citations"部分应注明：

- `^[raw/articles/...]` citations: 0%（格式未使用）
- `wikilinks`: 实际数量（实际 provenance）
- EXCESS INFERRED 是 wikilink 型 wiki 的预期警告，不代表真实质量问题

## 相关实体

- [[entities/wiki-evolver]]
- [[entities/agent-skill-writing]]
- [[entities/agent-skill-writing-evaluation]]

## 深度分析

### Shell vs Python 的角色分工

| 任务 | Shell | Python |
|------|-------|--------|
| 数量级确认 | ✓ grep/wc/cat | - |
| 细粒度分解 | - | ✓ regex/loops |
| Cross-validation | ✓ 两边必须一致 | ✓ 两边必须一致 |

Shell 用于快速确认数量级（例如：确认有 1000+ 还是 100+ 个 orphans），Python 用于细粒度分析（每个 entity 的具体链接关系）。**永远先跑 Shell，再跑 Python，两者结果必须一致才可信**。

### 常见错误的根本原因

1. **regex 字符类放反斜杠**: `[^|\]]` 会在字符类里引入一个实际的反斜杠字符，导致匹配行为不可预测
2. **忘记 strip `|`**: `target|display` 如果不 split('|')[0]，目标会变成 `"target|display"` 而找不到匹配
3. **混淆 source 和 target**: in_counts 的 source 是 `entities/` 下的文件，target 是 all_names 里的所有实体（entities + concepts + queries + comparisons）
4. **citation regex 里的 `[^...]`**: `[^raw]` 在字符类里是"非 r 非 a 非 w"，不是"非 raw 字符串"，导致匹配失败

## 关联实体

**上游依赖**:
- [[entities/wiki-evolver]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-skill-writing]] — 具体应用场景

**平行协作**:
- [[entities/agent-skill-writing-evaluation]] — 替代/补充方案

## 所属 MOC

- [[moc/ai-skill-design|Ai Skill Design]]
