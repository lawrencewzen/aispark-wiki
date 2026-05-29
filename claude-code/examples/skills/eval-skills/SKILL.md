> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: eval-skills
description: "审计当前项目中所有技能的 frontmatter 完整性、effort 级别适当性、allowed-tools 范围及内容质量，并生成带 effort 级别建议的评分报告。适合在新项目上手时、发布前审查技能质量、或为现有技能库首次添加 effort 字段时使用。"
allowed-tools: Read Glob Bash
argument-hint: "[path — 默认: .claude/skills/]"
effort: medium
---

# 技能评估器

发现项目中所有技能，按 6 项标准打分，并根据内容分析推断合适的 `effort` 级别。

## 使用场景

- 新项目：运行一次以建立质量基线
- 提交技能到团队仓库之前
- 从其他项目批量导入技能之后
- 首次添加 `effort` 字段时（v2.1.80+）

## 审计范围

所有位于以下路径的 `SKILL.md` 文件和扁平 `.md` 文件：
- `.claude/skills/**`
- `~/.claude/skills/**`（按需请求）
- 通过参数传入的任意路径：`/eval-skills ./my-skills-dir`

---

## 前置检查：使用官方验证器

手动打分前，先运行官方 CLI 验证器——它能在几秒内发现结构性问题：

```bash
# 安装（一次性）
uv tool install skills-ref

# 验证某个技能目录
skills-ref validate ./my-skill

# 验证项目中所有技能
find .claude/skills -name "SKILL.md" -exec dirname {} \; | xargs -I{} skills-ref validate {}
```

`skills-ref` 通过后，继续进行以下质量评分。

---

## 评分标准（每个技能满分 14 分）

| # | 标准 | 满分 | 检查内容 |
|---|-----------|-----|-----------------|
| 1 | **name** | 1 | 存在、小写、仅含连字符，且与目录名一致 |
| 2 | **description** | 2 | 存在 + 包含"Use when"/"when to"/触发短语 |
| 3 | **allowed-tools** | 2 | 存在 + 范围不过宽（只读操作时不应无限制使用 Bash） |
| 4 | **effort** | 3 | 存在（1分）+ 与内容匹配（2分，基于推断） |
| 5 | **内容结构** | 4 | 有 Purpose/When 部分（1），有示例/用法（1），有清晰工作流（1），无占位文本（1） |
| 6 | **加分项** | +2 | 存在 argument-hint（1），元数据含版本/作者（1） |

**合法的 frontmatter 字段**（agentskills.io 规范 + Claude Code 扩展）：
- `name`、`description`、`allowed-tools`、`license`、`compatibility`、`metadata` — agentskills.io 规范
- `effort`、`argument-hint`、`disable-model-invocation` — Claude Code 扩展
- `model` — Claude Code 扩展：覆盖本次技能调用所用模型

> **注意**：`tags`、`category`、`keywords`、`context`、`agent`、`usage`、`args` 不是 Claude Code 支持的 frontmatter 字段，运行时会忽略。审计时请标记并删除。

**`allowed-tools` 格式**：空格分隔的字符串，而非 YAML 列表。`Read Bash Grep` 是正确写法；`[Read, Bash, Grep]` 是错误写法，可能无法解析。

**评分阈值：**
- ✅ 良好：≥11/14（≥80%）
- ⚠️ 待改进：8–10/14（60–79%）
- ❌ 需修复：<8/14（<60%）

---

## Effort 级别推断引擎

针对每个技能，分析 description 和内容，根据以下信号进行分类：

### `low` — 机械性执行，无需设计决策

信号：
- 动词：commit、push、sync、scaffold、generate（基于模板）、format、rename、bump、wrap、convert
- 无需推理：顺序执行步骤、模板实例化、数据获取
- allowed-tools：仅 Bash，或仅 Read
- 不生成子智能体
- 工作流简短（<5 步）

示例：`/commit`、`/release-notes`、`/scaffold`、`/sync`、`/format`

### `medium` — 有限范围内的分析与分类

信号：
- 动词：review、triage、analyze、categorize、suggest、evaluate（单文件或有限范围）
- 需要模式识别，但不涉及架构级推理
- allowed-tools：Read + Grep + Bash 组合
- 可能生成 1-2 个子智能体，但范围预定义
- 产出结构化输出（表格、分类列表）

示例：`/code-review`（单个 PR）、`/issue-triage`、`/dependency-audit`、`/test-coverage`

### `high` — 设计决策、对抗性推理、跨系统分析

信号：
- 动词：architect、redesign、threat-model、audit（安全）、orchestrate（多智能体）、score、assess trade-offs
- 需要推理边界条件、攻击向量或全系统影响
- allowed-tools：宽泛访问权限（Read + Write + Bash + 外部工具）
- 生成多个子智能体或使用并行执行
- 产出含明确不确定性或权衡分析的报告
- 内容关键词："security"、"architecture"、"adversarial"、"pipeline"、"threat"、"design decision"

示例：`/security-audit`、`/architecture-review`、`/cyber-defense`、`/eval-agents`

### 不匹配标记

若技能已设置 `effort:` 但推断级别不同，则标记：
> ⚠️ Effort 不匹配：声明为 `low`，推断为 `high` —— 该技能生成 4 个子智能体并执行安全分析

---

## 执行步骤

### 第 1 步 — 发现

```bash
# 查找所有 SKILL.md 文件
find .claude/skills -name "SKILL.md" 2>/dev/null

# 查找扁平技能文件
find .claude/skills -maxdepth 1 -name "*.md" ! -name "README*" 2>/dev/null

# 若提供了参数，则使用该路径
```

### 第 2 步 — 解析每个技能

对找到的每个技能文件：
1. 读取完整文件
2. 提取 YAML frontmatter（第一个 `---` 与第二个 `---` 之间）
3. 解析：name、description、allowed-tools、effort、argument-hint、model、metadata
4. 记录各字段是否存在
5. 读取正文内容，分析结构

### 第 3 步 — 评分与推断

对每个技能应用上述评分标准：
- 检查 frontmatter 字段
- 评估 description 质量（是否回答了"何时使用"？是否在 1024 字符以内？）
- 评估 allowed-tools 范围（只需 Read 时是否使用了 Bash？是否尽可能用通配符限定工具范围？）
- 根据内容分析推断 effort 级别
- 对比推断值与已声明的 effort（如已设置）
- 评估内容结构（扫描"When to Use"、"Purpose"、"Example"、"Workflow"等章节）

### 第 4 步 — 输出

生成结构化报告：

```
# 技能审计 — [项目名称或路径]
日期：[今天] | 扫描：N 个技能

## 汇总
| 状态 | 数量 |
|--------|-------|
| ✅ 良好（≥80%） | N |
| ⚠️ 待改进（60–79%） | N |
| ❌ 需修复（<60%） | N |

**Effort 覆盖率**：N/N 个技能已设置 effort 字段

---

## 各技能结果

### [skill-name] — [分数]/14 [✅/⚠️/❌]

| 标准 | 得分 | 备注 |
|-----------|-------|-------|
| name | ✅ 1/1 | — |
| description | ⚠️ 1/2 | 缺少"Use when"短语 |
| allowed-tools | ✅ 2/2 | 范围合理 |
| effort | ❌ 0/3 | 缺失 — 建议：high |
| 内容结构 | ⚠️ 2/4 | 无示例章节 |

**Effort 推断**：`high` —— 该技能执行对抗性推理的安全分析
  信号：内容含"threat"、"attack surface"、"vulnerability scoring"；生成 4 个智能体

**优先修复项**（按影响排序）：
1. 在 frontmatter 中添加 `effort: high`
2. 在 description 中添加"Use when"
3. 添加具体用法示例章节

---
```

所有技能审计完毕后，打印**修复汇总**——列出所有缺失 effort 字段及建议值，可直接复制粘贴。

---

## 修复汇总格式

最后，打印一个可直接使用的补丁块，涵盖所有缺失或不匹配的 effort 字段：

```
## 建议的 effort 字段（可直接复制粘贴）

skill-name-1: effort: low     # 机械性脚手架
skill-name-2: effort: high    # 安全分析，生成智能体
skill-name-3: effort: medium  # 代码审查，有限范围
```

以及一行统计：`N 个技能需要添加 effort 字段 · N 个不匹配 · N 个缺少 allowed-tools`
