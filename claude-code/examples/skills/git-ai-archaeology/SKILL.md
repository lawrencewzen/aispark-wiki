> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: git-ai-archaeology
description: "分析 git 仓库中 AI 配置的演进历程。适用于梳理 AI 采用历史、追溯配置文件首次引入时间、按月统计提交频率，或识别项目 AI 工具链的成熟度阶段。"
allowed-tools: Write Read Bash
effort: medium
---

# git-ai-archaeology

对 git 仓库中 AI 配置的演进历程进行完整分析。找出每个 AI 配置文件的创建时间、AI 配置相关提交的逐月变化趋势、哪些 PR 推动了演进，并识别成熟度阶段。

**输出**：单个文件 `{output_dir}/{slug}-git-archaeology.md`

## 预期输入

```
/git-ai-archaeology repo_path=/path/to/repo [output=./talks/slug] [slug=talk-name] [since=2025-01-01]
```

- `repo_path`：目标 git 仓库的绝对路径（必填）
- `output`：输出目录（默认：`./talks`）
- `slug`：输出文件名（默认：仓库文件夹名称）
- `since`：分析起始日期（默认：仓库第一次提交时间）

## 工作流

1. **验证仓库**：确认路径存在且为 git 仓库
2. **全局指标**：总提交数、发布次数、贡献者数量、时间跨度
3. **第 1 节 — 首次提交**：找出关键 AI 配置路径的创建日期
4. **第 2 节 — 按月分布**：按 AI 配置关键词过滤的提交统计
5. **第 3 节 — 重要 PR**：提取并分类重要的 AI 配置提交
6. **第 4 节 — CHANGELOG**：若存在 CHANGELOG.md，提取含 AI 提及的发布记录
7. **第 5 节 — 阶段划分**：综合分析演进阶段
8. **保存**输出文件

---

## 第 1 步：验证与全局指标

```bash
# 验证是否为 git 仓库
git -C {repo_path} rev-parse --git-dir

# 全局指标
git -C {repo_path} log --oneline | wc -l                                    # 总提交数
git -C {repo_path} tag --sort=version:refname | wc -l                       # 总发布次数
git -C {repo_path} shortlog -sn --no-merges | wc -l                         # 贡献者数量
git -C {repo_path} log --pretty=format:"%ad" --date=short | tail -1         # 首次提交
git -C {repo_path} log --pretty=format:"%ad" --date=short | head -1         # 最近提交
git -C {repo_path} log --merges --oneline | wc -l                           # 已合并 PR 数
```

---

## 第 2 步：第 1 节 — 各 AI 配置路径的首次提交

对每个路径，使用 `--diff-filter=A` 找到原始提交：

```bash
# 待分析路径 — 根据仓库实际情况调整
PATHS=(
  "CLAUDE.md"
  ".claude"
  ".claude/commands"
  ".claude/agents"
  ".claude/hooks"
  ".claude/skills"
  ".claude/rules"
  ".agents"
  ".cursor"
  "doc/knowledge-base.md"
  "doc/guides/ai-instructions"
  "doc/guides/ai-review"
)

for path in "${PATHS[@]}"; do
  git -C {repo_path} log --diff-filter=A --follow \
    --format="%ad | %H | %s" --date=short \
    -- "$path" | tail -1
done
```

根据结果构建第 1 节表格。跳过无输出的路径（即该仓库中不存在的路径）。

同时构建 ASCII 时间线：
```
{date} ─── {path} ─── {message}
```
按时间顺序排列。

---

## 第 3 步：第 2 节 — AI 配置提交的按月分布

按 AI 配置相关关键词过滤提交：

```bash
# 含 AI 配置关键词的所有提交
git -C {repo_path} log --format="%H %s" | \
  grep -iE "(claude|feat.ai|docs.ai|tech.ai|mcp|skill|hook|agent|llm|prompt)" \
  > /tmp/ai_commits_filtered.txt

# 按月统计 AI 配置提交数
git -C {repo_path} log --format="%ad %H" --date=format:"%Y-%m" | \
  while read month hash; do
    if grep -q "$hash" /tmp/ai_commits_filtered.txt; then
      echo "$month"
    fi
  done | sort | uniq -c
```

更直接的替代方案：

```bash
git -C {repo_path} log --format="%ad %s" --date=format:"%Y-%m" | \
  grep -iE " (feat|fix|docs|tech|chore|refactor)\(ai\)|claude|mcp.*server|\.claude/|skill|hook.*security|guardrail" | \
  awk '{print $1}' | sort | uniq -c
```

按月计算：
- AI 配置提交数
- 占当月总提交的百分比（与各类别月度总量交叉对比）
- 背景说明（如属于显著时期）

构建 ASCII 分布图（横向或纵向柱状图）。

---

## 第 4 步：第 3 节 — 重要 PR 与提交

### 3.1 — feat(ai): / docs(ai): / tech(ai): 提交

```bash
git -C {repo_path} log --format="%ad | %H | %s" --date=short | \
  grep -iE "\(ai\)|\(mcp\)|\[ai\]"
```

### 3.2 — MCP Server 集成

```bash
git -C {repo_path} log --format="%ad | %H | %s" --date=short | \
  grep -iE "mcp|serena|grepai|perplexity|sonar|postgres.*mcp|cursor.*mcp"
```

### 3.3 — 技能、命令、钩子、智能体

```bash
git -C {repo_path} log --format="%ad | %H | %s" --date=short | \
  grep -iE "feat\(skill|feat\(hook|feat\(agent|feat\(command|feat\(dx\)|feat\(ci\)" | \
  grep -v "^$"
```

### 3.4 — 代码审查自动化

```bash
git -C {repo_path} log --format="%ad | %H | %s" --date=short | \
  grep -iE "review|code-review|pr.*auto|ci.*review"
```

---

## 第 5 步：第 4 节 — CHANGELOG 分析（如有）

```bash
# 检查 CHANGELOG.md 是否存在
ls {repo_path}/CHANGELOG.md

# 提取含 AI 提及的发布记录
grep -n "## \[" {repo_path}/CHANGELOG.md | head -30
```

读取 CHANGELOG 并构建表格：

| 版本 | 日期 | AI 相关内容 |
|------|------|------------|

仅列出含 AI 配置内容的发布（CLAUDE.md、MCP、智能体、技能、钩子、护栏、提示词等）。

---

## 第 6 步：第 5 节 — 演进阶段

分析收集到的数据并识别成熟度阶段。典型模式：

| 阶段 | 特征 | 提交量 | 标签 |
|------|------|--------|------|
| **阶段 1** | 基础配置、个人使用、无结构 | 少 | "配置是事后补充" |
| **阶段 2** | 文档化、知识库、首个 MCP | 增长 | "配置即文档" |
| **阶段 3** | 基础设施：技能/钩子/规则/MCP 栈 | 激增 | "配置即基础设施" |
| **阶段 4** | 工程化：测试、CI、护栏、模块化 | 密集 | "配置即工程实践" |

根据数据实际情况调整阶段划分。

识别**主要拐点**：AI 配置提交量激增的月份。

计算"近期与历史"比率（例如："过去 2 个月占 AI 配置提交总量的 81%"）。

---

## 输出格式：{slug}-git-archaeology.md

```markdown
# Git 考古 — AI 配置演进：{slug}

**来源**：仓库 `{repo_path}` 的 git 历史（{total_commits}+ 次提交，{total_releases}+ 次发布）
**方法**：`git log --diff-filter=A` 追溯首次提交、过滤月度分布、重要 PR
**最后更新**：{date}

---

## 第 1 节：各关键路径的首次提交

| 路径 | 创建日期 | 提交信息 | Hash |
|------|---------|---------|------|
{rows}

### 创建时间线

\```
{ascii_timeline}
\```

---

## 第 2 节：AI 配置提交的按月分布

| 月份 | AI 配置提交数 | 占总量百分比 | 背景 |
|------|-------------|------------|------|
{rows}

### 可视化

\```
{ascii_chart}
\```

**拐点**：{关于提交激增的洞察}

---

## 第 3 节：AI 工具链相关重要 PR 与提交

### 3.1 PR `feat(ai):` / `tech(ai):` / `docs(ai):`

| 日期 | Hash | 信息 | 影响 |
|------|------|------|------|
{rows}

### 3.2 MCP Server 集成（按时间顺序）

| 日期 | MCP Server | Hash / PR | 作用 |
|------|-----------|-----------|------|
{rows}

### 3.3 技能、命令、钩子、智能体

| 日期 | Hash | 信息 | 分类 |
|------|------|------|------|
{rows}

### 3.4 代码审查自动化

| 日期 | Hash | 信息 |
|------|------|------|
{rows}

---

## 第 4 节：CHANGELOG 各发布版本中的 AI 提及

{若有 CHANGELOG 则填充该节，否则填"不适用"}

---

## 第 5 节：演进阶段

### 基于证据的时间线

| 里程碑 | 精确 Git 日期 | Git 证据 |
|--------|-------------|---------|
{rows}

### {N} 个演进阶段

#### 阶段 1：{标签}（{时期}）— {n} 次提交
{描述}

#### 阶段 2：{标签}（{时期}）— {n} 次提交
{描述}

#### 阶段 3：{标签}（{时期}）— {n} 次提交
{描述}

#### 阶段 4：{标签}（{时期}）— {n} 次提交
{描述}

### 关键洞察

{总结段落：主要拐点、近期与历史比率、数据揭示的项目 AI 成熟度。}

---
*由 git-ai-archaeology 生成 — {date}*
*仓库：{repo_path} | {total_commits} 次提交 | {total_releases} 次发布*
```

---

## 重要规则

- **只读**：不执行任何修改仓库状态的 git 命令
- **先验证再断言**：git 中找不到的日期 = 注明"未验证"
- **适配路径**：第 1 节的路径必须过滤为该仓库实际存在的路径
- **可扩展关键词**：若仓库使用不同规范（如 `feat[ai]` 而非 `feat(ai)`），相应调整 grep 模式
- **第 4 节可选**：若无 CHANGELOG.md 或无 AI 提及，注明"不适用"并跳转第 5 节
- **自适应阶段**：4 个阶段是常见模式，不是规定 — 2 个或 6 个阶段同样有效

## 反模式

- 捏造 git 中未找到的数据
- 数字取整但不注明
- 分析该仓库中不存在的路径
- 将重命名提交误判为创建提交
- 省略"平淡"的月份（0 次 AI 配置提交同样说明问题）

## 验证清单

- [ ] 仓库已验证且可读
- [ ] 第 1 节：仅包含该仓库实际存在的路径
- [ ] 第 2 节：分布覆盖仓库完整时间跨度
- [ ] 第 3 节：提交按时间顺序排列，包含 hash
- [ ] 第 4 节：无 CHANGELOG 时已干净跳过
- [ ] 第 5 节：阶段基于数据而非模板
- [ ] 输出文件已保存
