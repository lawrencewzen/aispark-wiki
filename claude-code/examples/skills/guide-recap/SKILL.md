> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: guide-recap
description: "将 CHANGELOG 条目转换为社交媒体内容（LinkedIn、Twitter/X、Newsletter、Slack），支持法语和英语。可在发布后或每周使用，从指南更新中生成发布说明、公告、社交媒体帖子或摘要。"
allowed-tools: Read Bash
argument-hint: "<latest|vX.Y.Z|week [YYYY-MM-DD]> [--interactive] [--format=linkedin|twitter|newsletter|slack] [--lang=fr|en] [--save]"
effort: low
---

# 指南摘要

从 CHANGELOG.md 条目生成社交媒体内容。默认产出 8 项内容（4 种格式 × 2 种语言）。

## 使用场景

- 运行 `/release` 后创建社交媒体公告
- 每周汇总多个发布版本
- 在 LinkedIn、Twitter/X、Newsletter 或 Slack 上发布之前

## 用法

```
/guide-recap latest              # 最新发布版本
/guide-recap v3.20.5             # 特定版本
/guide-recap week                # 当前周（周一至今天）
/guide-recap week 2026-01-27     # 特定周（周一至周日）
```

### 参数标志

| 标志 | 效果 | 默认值 |
|------|--------|---------|
| `--interactive` | 引导模式：选择角度、受众、亮点 | 关（自动起草） |
| `--format=X` | 单一格式：`linkedin`、`twitter`、`newsletter`、`slack` | 全部 4 种格式 |
| `--lang=X` | 单一语言：`fr`、`en` | 法语 + 英语 |
| `--save` | 将输出保存到 `[project-docs]/social-posts/` | 仅显示 |
| `--force` | 即使只有维护性条目也强制生成 | 跳过低分内容 |

## 工作流（7 个步骤）

### 步骤一：解析输入

解析 `$ARGUMENTS` 以确定模式：

| 输入 | 模式 | 目标 |
|-------|------|--------|
| `latest` | 单一版本 | `[Unreleased]` 之后第一个 `## [X.Y.Z]` |
| `vX.Y.Z` 或 `X.Y.Z` | 单一版本 | 精确版本匹配 |
| `week` | 周范围 | 当前周的周一 -> 今天 |
| `week YYYY-MM-DD` | 周范围 | 该周一 -> 次周日 |

如果没有参数或参数无效，显示用法说明并退出。

### 步骤二：提取 CHANGELOG 条目

从项目根目录读取 `CHANGELOG.md`。

**单一版本：**
1. 找到匹配 `## [{version}]` 的行
2. 提取到下一个 `## [` 行之前的所有内容
3. 解析 `### Added`、`### Changed`、`### Fixed` 部分

**周范围：**
1. 收集日期在范围内的所有 `## [X.Y.Z] - YYYY-MM-DD` 条目
2. 解析所有匹配版本的所有部分

**错误：版本未找到** -> 列出最近 5 个版本，建议使用 `latest`。
**错误：该周没有条目** -> 显示最后一次发布日期，建议使用该版本。

### 步骤三：分类条目

对每个顶级条目（`###` 下的一级项目符号），分配类别：

| 类别 | 权重 | 识别方式 |
|----------|--------|-----------|
| `NEW_CONTENT` | 3 | 新文件、新章节、新图表、新测验题 |
| `GROWTH_METRIC` | 2 | 行数增长、条目数量变化 |
| `RESEARCH` | 1 | 资源评估、外部来源集成 |
| `FIX` | 1 | 在 `### Fixed` 下，修正内容 |
| `MAINTENANCE` | 0 | README 更新、徽章同步、首页同步、数量更新 |

详细分类规则参见 `references/changelog-parsing-rules.md`。

### 步骤四：转化为用户价值

应用 `references/content-transformation.md` 中的映射：

- 技术语言 -> 用户收益
- 提取具体数字
- 标注命名来源
- 聚合相关条目

对照 `references/tone-guidelines.md` 的 DO/DON'T 清单进行验证。

### 步骤四b：交互模式（仅 --interactive）

如果设置了 `--interactive` 标志，在步骤四和步骤五之间插入：

1. 按分数显示候选亮点：
   ```
   亮点（按分数排序）：
   [14] 4 张新 ASCII 图表 (16 -> 20)       [NEW_CONTENT]
   [ 9] 30 道新测验题 (227 -> 257)         [NEW_CONTENT]
   [ 6] Docker 沙盒隔离指南               [NEW_CONTENT]
   [ 1] README 已更新                      [MAINTENANCE]
   ```

2. 询问角度：
   - 自动（得分最高的作为钩子）
   - 用户选择特定条目作为钩子
   - 自定义角度（用户提供主题）

3. 询问目标受众：
   - `devs`（技术深度）
   - `tech-leads`（影响力聚焦）
   - `general`（通俗语言）
   - `all`（默认，均衡）

4. 询问主要亮点：
   - 自动（最高分）
   - 用户从列表中选择

5. 确认选择并继续步骤五。

### 步骤五：评分和筛选

计算每个条目的分数：

```
score = (category_weight * 3)
      + (has_number * 2)
      + (named_source * 1)
      + (new_file * 1)
      + (min(impact_files, 3))
      + (breaking * 2)
```

按分数选择前 3-4 个条目。最高分 = 钩子行。

**如果所有分数 < 3**：输出"此版本不建议生成社交内容。使用 `--force` 强制生成。"并退出（除非使用 `--force`）。

### 步骤六：生成内容

对每种请求的格式（默认：全部 4 种）和语言（默认：两种）：

1. 从 `assets/` 读取对应模板
2. 使用评分和转化后的条目填充模板字段
3. 应用 tone-guidelines.md 质量清单

**链接：**

| 格式 | 链接目标 |
|--------|-------------|
| LinkedIn | 着陆页 URL |
| Twitter | GitHub 仓库 URL |
| Newsletter | 两者（着陆页 + GitHub） |
| Slack | GitHub 仓库 URL |

**URL：**
- 着陆页：`https://{DOMAIN}/`
- GitHub：`https://github.com/{OWNER}/{REPO}`

### 步骤七：输出

在代码围栏中显示每条生成的帖子，按格式和语言标注：

```
## LinkedIn (FR)

```text
[内容]
`` `

## LinkedIn (EN)

```text
[内容]
`` `

## Twitter/X (FR)

```text
[内容]
`` `

...
```

如果设置了 `--save` 标志：将所有输出写入 `[project-docs]/social-posts/YYYY-MM-DD-vX.Y.Z.md`（版本模式）或 `[project-docs]/social-posts/YYYY-MM-DD-week.md`（周模式）。如果 `[project-docs]/social-posts/` 目录不存在则创建。

## 错误处理

| 错误 | 响应 |
|-------|----------|
| 无参数 | 显示用法说明块 |
| 参数无效 | 显示带示例的用法说明块 |
| 版本未找到 | 列出最近 5 个版本，建议使用 `latest` |
| 该周无条目 | 显示最后一次发布日期，建议使用对应版本 |
| 所有条目为 MAINTENANCE（分数 0）| "不建议生成社交内容。使用 `--force` 覆盖。" |
| CHANGELOG.md 未找到 | "项目根目录中未找到 CHANGELOG.md。" |

## 参考文件

- `references/tone-guidelines.md` - DO/DON'T 规则、emoji 用量、语言风格
- `references/changelog-parsing-rules.md` - CHANGELOG 格式、提取方式、评分算法
- `references/content-transformation.md` - 技术语言 -> 用户价值映射（30+条）
- `assets/linkedin-template.md` - 约 1300 字符，钩子 + 要点 + CTA + 话题标签
- `assets/twitter-template.md` - 280 字符单条或 2-3 条推文线程
- `assets/newsletter-template.md` - 约 500 词，结构化章节
- `assets/slack-template.md` - 简洁，富含 emoji，Slack 格式
- `examples/version-output.md` - v3.20.5 的完整示例输出
- `examples/week-output.md` - 2026-01-27 周的完整示例输出

## 使用技巧

- 运行 `/release` 后立即执行 `/guide-recap latest` 准备社交媒体帖子
- 前几次使用 `--interactive` 以了解评分机制
- 只需要某个特定输出时使用 `--format=linkedin --lang=fr`
- `--save` 输出通过 `[project-docs]/` 约定被 gitignore
- 发布前请审阅并个性化处理（这些是草稿，不是最终文案）
