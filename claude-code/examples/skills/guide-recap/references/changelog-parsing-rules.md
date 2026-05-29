> 📚 **AI Spark Wiki** · Claude Code 知识库

# CHANGELOG 解析规则

如何从 `CHANGELOG.md` 中提取并分类条目，用于生成社交内容。

## CHANGELOG 格式

本项目遵循 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) 规范。

```
## [Unreleased]

## [X.Y.Z] - YYYY-MM-DD

### Added
- **Title**: Description
  - Sub-detail 1
  - Sub-detail 2

### Changed
- **Title**: Description

### Fixed
- **Title**: Description

---
（水平分隔线用于分隔同一版本内的分组条目）
```

## 提取方式

### 按版本提取（`/guide-recap v3.20.5`）

1. 读取 CHANGELOG.md
2. 找到匹配 `## [3.20.5]` 的行
3. 提取直到下一个 `## [` 行之前的全部内容
4. 解析所有 `### Added/Changed/Fixed` 小节

### 最新版本（`/guide-recap latest`）

1. 读取 CHANGELOG.md
2. 跳过 `## [Unreleased]`
3. 提取第一个 `## [X.Y.Z]` 块（即最新发布版本）

### 按周提取（`/guide-recap week` 或 `/guide-recap week 2026-01-27`）

1. 确定日期范围：
   - 不带日期的 `week`：本周一 -> 今天
   - `week YYYY-MM-DD`：该周一 -> 下周日
2. 读取 CHANGELOG.md
3. 收集日期落在范围内的所有 `## [X.Y.Z] - YYYY-MM-DD` 条目
4. 合并所有版本的条目

## 条目结构

`### Added/Changed/Fixed` 下的每个顶级列表项是一个**条目**。解析字段如下：

| 字段 | 来源 | 示例 |
|-------|--------|---------|
| `title` | `- **` 后的加粗文本 | `Visual Reference` |
| `description` | 同行 `:` 或 `--` 后的文本 | `4 new high-value ASCII diagrams (16 -> 20 total)` |
| `sub_items` | 下方缩进列表 | 详细内容列表 |
| `section` | 父级 `###` 标题 | `Added`、`Changed`、`Fixed` |
| `files` | 描述中的文件名/路径 | `guide/ultimate-guide.md`、`machine-readable/reference.yaml` |
| `source` | URL 或具名引用 | `Pat Cullen's Final Review Gist` |
| `metrics` | 描述中的数字 | `227 -> 257`、`+522 lines`、`4 new` |
| `score` | 评估分数（如有） | `Score: 4/5` |

## 类别分类

每个条目对应唯一类别，并附带权重：

| 类别 | 权重 | 模式 |
|----------|--------|---------|
| `NEW_CONTENT` | 3 | 新增指南章节、新文件、新图表、新测验题 |
| `GROWTH_METRIC` | 2 | 行数增长、模板数量、测验题数量变化 |
| `RESEARCH` | 1 | 资源评估、引入外部来源 |
| `FIX` | 1 | 错误修复、纠正、准确性提升 |
| `MAINTENANCE` | 0 | README 更新、徽章更新、数量同步、落地页同步 |

### 分类规则

1. 若条目创建了新 `.md` 文件或新章节 -> `NEW_CONTENT`
2. 若条目包含带数字的 `-> `（增长） -> `GROWTH_METRIC`
3. 若条目引用了带评估分数的外部来源 -> `RESEARCH`
4. 若条目位于 `### Fixed` 下 -> `FIX`
5. 若条目仅更新数量、徽章或同步操作 -> `MAINTENANCE`
6. 若条目的子项包含实质性内容 -> 提升一个级别
7. 模糊时优先选择权重更高的类别

### 示例

```
"4 new ASCII diagrams (16 -> 20)"           -> NEW_CONTENT（新图表）
"30 New Quiz Questions (227 -> 257)"        -> NEW_CONTENT（新题目）
"Quiz badge updated (227 -> 257)"           -> MAINTENANCE（徽章同步）
"Guide line count: 15,771 -> 16,293"        -> GROWTH_METRIC（增长）
"Score: 4/5 - Docker Sandboxes"             -> RESEARCH（评估）
"Fixed 14 -> 15 categories in landing"      -> FIX（纠正）
"README.md: Added Visual Reference to table" -> MAINTENANCE（导航更新）
```

## 评分算法

对每个条目计算：

```
score = (category_weight * 3)
      + (has_number * 2)
      + (named_source * 1)
      + (new_file * 1)
      + (min(impact_files, 3))
      + (breaking * 2)
```

| 因子 | 值 | 检测方式 |
|--------|-------|-----------|
| `category_weight` | 0-3 | 来自上方类别表 |
| `has_number` | 0 或 1 | 条目包含数字变化（`N -> M`、`+N lines`、`N new`） |
| `named_source` | 0 或 1 | 条目注明了具体人员或外部来源 |
| `new_file` | 0 或 1 | 条目提到创建了新文件（`NEW`、新 `.md`） |
| `impact_files` | 0-3 | 涉及的不同文件数量（上限 3） |
| `breaking` | 0 或 1 | 条目位于 `### Breaking` 下或提到破坏性变更 |

### 分数解读

| 分数 | 动作 |
|-------|--------|
| 10+ | 主要亮点（钩子行） |
| 6-9 | 次要亮点（列表项） |
| 3-5 | 有空间时纳入 |
| 0-2 | 跳过（维护性噪音） |

选取分数最高的 3-4 个条目。若所有分数均低于 3，标记为"不建议生成社交内容"。

## 周报聚合规则

生成包含多个版本的周报时：

1. 对日期范围内所有版本的条目统一评分
2. 去重：同一主题出现在多个版本时，保留分数最高的条目
3. 在周报开头注明版本数量：`本周 X 个发布`
4. 标题使用日期范围，而非单个版本号
