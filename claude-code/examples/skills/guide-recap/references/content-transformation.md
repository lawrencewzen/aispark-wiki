> 📚 **AI Spark Wiki** · Claude Code 知识库

# 内容转化

将技术性 CHANGELOG 语言映射为面向用户的社交价值内容。所有输出均须遵守 tone-guidelines.md 中的规范。

## 转化原则

```
技术事实（CHANGELOG）-> 用户价值（社交帖子）
```

不得凭空创造收益描述。每一条转化都必须能追溯到 CHANGELOG 中的具体条目。

## 映射表

### 新增内容

| 技术描述（CHANGELOG） | 社交内容（用户价值） |
|---|---|
| `N new ASCII diagrams (X -> Y total)` | `视觉型学习者？新增 N 张 [主题] 图表` |
| `N New Quiz Questions (X -> Y total)` | `测测你的 Claude Code 知识：新增 N 题，涵盖 [主要类别]` |
| `New [guide-name].md (N lines)` | `新指南：[用通俗语言描述主题]` |
| `New section: [Section Name] (~N lines)` | `[你现在能学到/做到什么]：[通俗描述]` |
| `New workflow: [name]` | `手把手：如何用 Claude Code [执行动作]` |
| `Enhanced [command/agent] (+N lines)` | `[命令] 现已支持 [新能力]` |
| `N new entries in reference.yaml` | 跳过（内部索引，无用户价值） |

### 研究与来源

| 技术描述（CHANGELOG） | 社交内容（用户价值） |
|---|---|
| `Score: 5/5 - [Source]` | `来自 [作者] 的关键发现：[核心洞察]` |
| `Score: 4/5 - [Source]` | `来自 [作者] 的研究：[实用结论]` |
| `Score: 3/5 - [Source]` | `[作者] 确认：[相关发现]` |
| `Source: [URL] ([Author], [Date])` | `基于 [作者] 的研究` |
| `Competitive Analysis: N Gaps from [source]` | `识别出 N 个进阶模式：[前 2-3 个名称]` |
| `Resource evaluation: [file]` | 跳过（内部流程，不面向用户） |
| `Fact-checked: N/N claims verified` | 可作为可信度信号提及：`全部 N 条声明已核实` |

### 增长指标

| 技术描述（CHANGELOG） | 社交内容（用户价值） |
|---|---|
| `Guide line count: X -> Y (+N lines)` | `本 [周/版本] 新增 +N 行文档` |
| `X -> Y total [items]` | `现已达 Y [项]（原为 X）` |
| `+N lines: X -> Y` | `[功能] 扩展了 N 行 [内容类型]` |

### 修复与变更

| 技术描述（CHANGELOG） | 社交内容（用户价值） |
|---|---|
| `Fixed [technical issue]` | `已修正：[用户可见的修复内容]` |
| `[File]: Updated [field] (X -> Y)` | 除非用户可见，否则跳过 |
| `Landing synced` | 跳过（基础设施） |
| `Badge updated` | 跳过（基础设施） |

### 维护性更新（通常跳过）

| 技术描述（CHANGELOG） | 社交价值 |
|---|---|
| `README.md: Added [X] to table` | 跳过 |
| `Updated counts in [files]` | 跳过 |
| `Sync: [description]` | 跳过 |
| `reference.yaml: +N entries` | 跳过 |
| `CLAUDE.md: [update]` | 跳过 |

## 模式识别

### 需重点突出的数字

当条目包含数字变化时，提取并格式化：

```
"30 New Quiz Questions (227 -> 257)"
-> 数量: 30
-> 增长: "227 -> 257"
-> 重点: "新增 30 题" 或 "现已达 257 题"

"4 new ASCII diagrams (16 -> 20 total)"
-> 数量: 4
-> 增长: "16 -> 20"
-> 重点: "新增 4 张图表" 或 "现已达 20 张"

"+522 lines"
-> 数量: 522
-> 重点: "新增 +522 行内容"
```

### 需标注的具名来源

提取作者姓名并给予适当归因：

```
"Pat Cullen's Final Review Gist" -> "来自 Pat Cullen 的生产工作流"
"Addy Osmani's 80% Problem"     -> "基于 Addy Osmani 的研究"
"Shen & Tamkin RCT"             -> "来自 Shen & Tamkin 的研究（Anthropic）"
"claudelog.com (InventorBlack)"  -> "由 claudelog.com 社区发现"
"Jude Gao (Vercel)"             -> "来自 Vercel 的基准测试（Jude Gao）"
```

### 主题聚合

当多个条目共享同一主题时，将其聚合：

```
关于代码审查的条目：
- "Multi-Agent PR Review"
- "Enhanced /review-pr command"
- "Enhanced code-reviewer agent"
-> 聚合："代码审查全面升级：多智能体工作流、反幻觉规则、严重性分类"

关于安全的条目：
- "Docker sandbox isolation"
- "Security 3-Layer Defense diagram"
- "Secret Exposure Timeline diagram"
-> 聚合："安全聚焦：沙盒隔离、防御层次、事件响应时间线"
```

## 版本 vs 周报框架

### 单一版本

```
FR: "Claude Code Ultimate Guide v3.20.5"
EN: "Claude Code Ultimate Guide v3.20.5"
```

### 周报（多个版本）

```
FR: "Cette semaine dans le guide (N releases)"
EN: "This week in the guide (N releases)"
```

### 周报（单一版本）

```
FR: "Cette semaine : guide v3.20.5"
EN: "This week: guide v3.20.5"
```
