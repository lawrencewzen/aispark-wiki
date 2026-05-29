> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: issue-triage
description: "三阶段 issue 积压管理工作流：审计、深度分析与已验证的分类操作。适用于分类 GitHub issue、整理 bug 报告、清理过期工单或检测重复 issue。参数：'all' 分析全部，issue 编号指定处理（如 '42 57'），'en'/'fr' 选择语言，无参数 = 仅审计。"
allowed-tools: Bash
effort: medium
---

# Issue 分类

面向维护者的三阶段工作流：自动审计所有开放 issue、按需通过并行智能体进行深度分析，以及已验证的分类操作（评论、标签、关闭）。

## 何时使用本技能

| 技能 | 用途 | 输出 |
|------|------|------|
| `/issue-triage` | 整理、分析并处理 issue 积压 | 分类表格 + 分析结果 + 已执行操作 |
| `/pr-triage` | 整理、审查并评论 PR 积压 | 分类表格 + 审查结果 + 已发布评论 |

**触发时机**：
- 手动触发：`/issue-triage` 或 `/issue-triage all` 或 `/issue-triage 42 57`
- 主动触发：检测到超过 10 个无标签的开放 issue，或存在超过 30 天未更新的过期 issue

---

## 语言

- 检查传入技能的参数
- 若为 `en` 或 `english` → 表格和摘要使用英文
- 若为 `fr`、`french` 或无参数 → 使用法语（默认）
- 注意：第 3 阶段的 GitHub 评论和标签**始终**使用英文（面向国际受众）

---

## 配置

整个工作流使用的阈值，可根据项目需要调整：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `staleness_days` | 30 | 无活动超过多少天后标记为过期 |
| `very_stale_days` | 90 | 无活动超过多少天后标记为严重过期 |
| `jaccard_threshold` | 60% | 将两个 issue 标记为重复的最低 Jaccard 相似度 |
| `closed_compare_count` | 20 | 用于重复检测的最近已关闭 issue 数量 |
| `open_limit` | 100 | 获取和分析的最大开放 issue 数量 |

---

## 前置条件

```bash
git rev-parse --is-inside-work-tree
gh auth status
```

若任一命令失败，则停止并说明缺少什么。

---

## 第 1 阶段 — 审计（始终执行）

### 数据收集（并行命令）

```bash
# 仓库标识
gh repo view --json nameWithOwner -q .nameWithOwner

# 开放 issue（排除 PR，限制 100 条）
gh issue list --state open --limit 100 \
  --json number,title,author,createdAt,updatedAt,labels,body,comments,assignees,milestone

# 最近已关闭 issue（用于重复检测）
gh issue list --state closed --limit 20 \
  --json number,title,body,labels,stateReason

# 开放 PR（正文用于交叉引用检测）
gh pr list --state open --limit 50 --json number,title,body

# 协作者列表（用于区分报告者类型）
gh api "repos/{owner}/{repo}/collaborators" --jq '.[].login'
```

**协作者列表降级方案**：若 `gh api .../collaborators` 返回 403/404：
```bash
# 从最近 10 个已合并 PR 中提取作者
gh pr list --state merged --limit 10 --json author --jq '.[].author.login' | sort -u
```
若仍无法确定，通过 `AskUserQuestion` 询问用户。

**注意**：`gh issue list --json comments` 中的 `comments` 字段返回的是数量，而非内容。第 2 阶段需单独获取完整内容：`gh issue view {num} --json comments`。

### 分析维度

对每个开放 issue 运行以下 6 个维度的分析：

#### 1. 分类

通过读取 `title` + `body` 前 200 个字符对每个 issue 进行分类：

| 类别 | 标签 | 判断标准 |
|------|------|----------|
| Bug | `bug` | 描述了异常行为、意外输出或崩溃 |
| 功能请求 | `enhancement` | 要求新功能 |
| 问题 / 支持 | `question` | 用户询问功能用法 |
| 文档 | `documentation` | 文档缺失或有误 |
| 超出范围 | `wontfix` | 明显超出项目边界 |
| 不明确 | `needs-info` | 正文为空或过于模糊无法分类 |

若正文为空 → 类别始终为"不明确"（切勿推测）。

#### 2. 与 PR 的交叉引用

扫描每个开放 PR 的正文，查找对 issue 编号的引用：
- 匹配模式：`fixes #N`、`closes #N`、`resolves #N`、`fix #N`、`close #N`（不区分大小写，`N` = issue 编号）
- 对已获取的 `body` 字段本地使用正则匹配，**不**发起额外 API 调用
- 若找到匹配：将 issue 标记为"PR 关联"并附上 PR 编号

#### 3. 通过 Jaccard 相似度检测重复

**算法（自包含，无需外部库）**：

对每个开放 issue，计算其与所有其他开放 issue 以及最近 20 个已关闭 issue 的 Jaccard 相似度。

```
步骤 1 — 规范化标题 + 正文前 300 个字符：
  - 将完整文本转为小写
  - 去除类别前缀："feat:"、"fix:"、"bug:"、"chore:"、"docs:"、"test:"、"refactor:"
  - 去除标点：.,!?;:'"()[]{}-_/\@#

步骤 2 — 分词：
  - 按空白字符分割
  - 去除停用词：the a an is in on to for of and or with this that it can not no be
  - 去除长度小于 3 的词元

步骤 3 — 计算 Jaccard：
  tokens_A = issue A 的词元集合
  tokens_B = issue B 的词元集合
  jaccard = |tokens_A ∩ tokens_B| / |tokens_A ∪ tokens_B|

步骤 4 — 标记：
  - 若 jaccard >= 0.60：标记为潜在重复
  - 报告："Similar to #N (Jaccard: 0.72)"
  - 保留较旧的 issue 为主版本；较新的为重复候选
```

Jaccard 在运行时使用已获取数据计算，第 1 阶段收集之外无需额外 API 调用。

#### 4. 风险分类

根据标题 + 正文中的信号，分配红色 / 黄色 / 绿色：

| 级别 | 颜色 | 判断标准 |
|------|------|----------|
| 严重 | 红色 | 安全漏洞、数据丢失、影响用户的回归问题、生产环境崩溃 |
| 需关注 | 黄色 | 缺少验证、性能下降、未记录的破坏性变更、不明确且超过 7 天无响应 |
| 正常 | 绿色 | 其他所有情况 |

#### 5. 过期状态

| 状态 | 判断标准 |
|------|----------|
| 活跃 | 30 天内有更新 |
| 过期 | 30–90 天无活动 |
| 严重过期 | 超过 90 天无活动 |

使用 `updatedAt` 字段。过期状态与评论数量无关——有评论但 `updatedAt` 较旧的 issue 仍视为过期。

#### 6. 建议操作

每个 issue 对应一条建议操作：

| 情况 | 操作 |
|------|------|
| 类别 = 不明确，正文为空 | 评论请求详细信息 |
| Jaccard >= 0.60 与已知 issue 相似 | 关闭为重复，并链接原始 issue |
| 严重过期且无负责人 | 评论询问状态，建议关闭 |
| 风险 = 红色 | 置顶到分类列表顶部，立即上报 |
| 类别 = 超出范围 | 附说明关闭 |
| PR 关联 | 无需操作（通过 PR 跟踪） |
| 正常 + 已有标签 | 无需操作 |

### 输出 — 分类表格

```
## 开放 Issue（{count} 个）

### 严重 — 立即处理（风险：红色）
| # | 标题 | 类别 | 报告者 | 开放天数 | 操作 |
|---|------|------|--------|----------|------|

### PR 关联（在开放 PR 中跟踪）
| # | 标题 | 类别 | PR | 开放天数 |
|---|------|------|----|----------|

### 活跃 Issue
| # | 标题 | 类别 | 标签 | 报告者 | 天数 | 操作 |
|---|------|------|------|--------|------|------|

### 重复候选
| # | 标题 | 相似于 | Jaccard | 操作 |
|---|------|--------|---------|------|

### 过期 Issue
| # | 标题 | 类别 | 最后活动 | 报告者 | 操作 |
|---|------|------|----------|--------|------|

### 摘要
- 开放总数：{N}
- 严重（红色）：{count}
- PR 关联：{count}
- 重复候选：{count}
- 过期（30–90 天）：{count}
- 严重过期（>90 天）：{count}
- 无标签：{count}
- 建议操作：{评论: N, 标签: N, 关闭: N}
```

0 个 issue → 显示 `No open issues.` 并停止。

**保护规则**（适用于所有阶段）：
- 未经明确用户确认，不得关闭协作者创建的 issue
- 不得重新标记已有标签的 issue（只添加缺失的标签）
- 若正文为空 → 在执行任何其他操作前，始终先请求详细信息
- 未经用户确认，不得自动关闭红色风险 issue

### 自动复制

展示分类表格后，使用平台对应的命令复制到剪贴板：

```bash
UNAME=$(uname -s)
if [ "$UNAME" = "Darwin" ]; then
  pbcopy <<'EOF'
{full triage tables}
EOF
elif command -v xclip &>/dev/null; then
  echo "{full triage tables}" | xclip -selection clipboard
elif command -v wl-copy &>/dev/null; then
  echo "{full triage tables}" | wl-copy
elif command -v clip.exe &>/dev/null; then
  echo "{full triage tables}" | clip.exe
fi
```

确认提示：`Triage tables copied to clipboard.`（英文）/ `Tableaux copiés dans le presse-papier.`（法文）

---

## 第 2 阶段 — 深度分析（按需）

### Issue 选择

**若已传入参数**：
- `"all"` → 所有有建议操作的 issue
- 编号（如 `"42 57"`）→ 仅指定的 issue
- 无参数 → 通过 `AskUserQuestion` 询问用户

**若无参数**，显示：

```
question: "你想深度分析哪些 issue？"
header: "深度分析"
multiSelect: true
options:
  - label: "全部（{N} 个有建议操作的 issue）"
    description: "为每个可操作 issue 启动并行分析智能体"
  - label: "仅严重问题（{M} 个红色风险 issue）"
    description: "聚焦于需立即处理的高风险 issue"
  - label: "重复候选（{K} 个 issue）"
    description: "通过完整正文和评论验证 Jaccard 相似度"
  - label: "仅过期（{J} 个过期 issue）"
    description: "决定哪些过期 issue 需关闭，哪些需重新跟进"
  - label: "跳过"
    description: "到此为止——仅审计"
```

若选择"跳过" → 结束工作流。

### 执行分析

对每个选定的 issue，通过 **Task 工具并行**启动分析智能体：

```
subagent_type: general
model: sonnet
prompt: |
  分析 GitHub issue #{num}："{title}"

  **元数据**：类别={category}，风险={risk}，开放天数={days}，标签={labels}
  **报告者**：@{author}（{collaborator? "协作者" : "外部用户"}）
  **负责人**：{assignees 或 "无"}

  **正文**：
  {body}

  **评论**（通过以下命令获取：gh issue view {num} --json comments）：
  {comments[].body — 总计截断至 5000 字符}

  **重复候选**：{jaccard_results 或 "未发现"}
  **关联 PR**：{pr_refs 或 "无"}

  任务：
  1. 验证第 1 阶段分配的类别（是否正确？若不正确，建议替代类别）
  2. 若为重复候选：确认或否认相似性，并说明理由
  3. 若为不明确/needs-info：明确指出缺少哪些信息
  4. 建议最合适的操作，若需要评论，提供完整评论文本
  5. 若为 Bug 或功能请求，估计修复所需工作量（XS/S/M/L/XL）

  返回结构化输出：
  ### 验证
  ### 重复分析
  ### 缺失信息
  ### 建议操作
  ### 工作量估计
```

**并行智能体不可用时的降级方案**：按顺序逐个分析 issue。通知用户：`Running sequential analysis (parallel agents not available).`

通过以下命令获取完整评论：
```bash
gh issue view {num} --json comments --jq '.comments[].body'
```

汇总所有报告，所有分析完成后展示摘要。

---

## 第 3 阶段 — 操作（强制验证）

### 草稿生成

对每个已分析的 issue，使用模板 `templates/issue-comment.md` 生成对应操作。

**3 种操作类型**：

| 类型 | 命令 | 适用场景 |
|------|------|----------|
| 评论 | `gh issue comment {num} --body-file -` | 需要信息、过期提醒、超出范围说明 |
| 标签 | `gh issue edit {num} --add-label "{label}"` | 类别明确的无标签 issue |
| 关闭 | `gh issue close {num} --reason "not planned"` | 重复、超出范围、严重过期 |

**规则**：
- 评论语言：**英文**（面向国际受众）
- 添加标签：仅使用仓库已有标签（通过 `gh label list` 获取）
- 关闭原因：超出范围/重复使用 `"not planned"`，仅在修复已合并时使用 `"completed"`
- 不得在用户未看到两份草稿的情况下同时发布评论并关闭 issue
- 关闭时始终附加评论（说明原因）

### 展示与验证

以如下格式**展示所有草稿操作**：

```
---
### 草稿 — Issue #{num}：{title}

**操作**：{评论 / 标签 / 关闭 + 评论}
**原因**：{一句话说明}

{如有评论，显示完整评论文本}

---
```

然后通过 `AskUserQuestion` 请求验证：

```
question: "这些操作已就绪。你想执行哪些？"
header: "执行分类操作"
multiSelect: true
options:
  - label: "全部（{N} 个操作）"
    description: "执行所有已草拟的分类操作"
  - label: "Issue #{x} — {title_truncated}（{action_type}）"
    description: "仅执行此操作"
  - label: "无"
    description: "取消——不执行任何操作"
```

（为每个 issue 生成一个选项，加上"全部"和"无"）

### 执行

对每个已验证的操作：

```bash
# 评论
gh issue comment {num} --body-file - <<'TRIAGE_EOF'
{comment}
TRIAGE_EOF

# 标签
gh issue edit {num} --add-label "{label}"

# 关闭并附评论
gh issue comment {num} --body-file - <<'TRIAGE_EOF'
{close comment}
TRIAGE_EOF
gh issue close {num} --reason "not planned"
```

确认每个操作：`Action executed on issue #{num}: {title}`

若选择"无" → `No actions executed. Workflow complete.`

---

## 边界情况

| 情况 | 处理方式 |
|------|----------|
| 0 个开放 issue | 显示 `No open issues.` 并停止 |
| 正文为空 | 类别 = 不明确，操作 = 请求详细信息，不得推测 |
| 协作者作为报告者 | 防止自动关闭，在表格中明确标记 |
| Jaccard 不确定（0.55–0.65） | 标记为"可能重复——请手动确认" |
| 标签不在仓库中 | 跳过标签操作，通知用户先创建该标签 |
| 工作流进行中 issue 已关闭 | 静默跳过，在摘要中注明 |
| `gh api .../collaborators` 返回 403/404 | 降级为最近 10 个已合并 PR 的作者 |
| 并行智能体不可用 | 顺序执行分析，通知用户 |
| 正文过长（>5000 字符） | 截断至 5000 字符并附 `[truncated]` 说明 |
| 已分配里程碑 | 在表格中显示，未经确认不得关闭里程碑 issue |

---

## 注意事项

- 始终通过 `gh repo view` 获取 owner/repo，不得硬编码
- 使用 `gh` CLI（而非 `curl` GitHub API），协作者列表除外
- `gh issue list --json comments` 中的 `comments` 仅为数量；完整内容需通过 `gh issue view {num} --json comments` 获取
- 未经用户在对话中明确验证，不得执行任何操作
- 草稿操作必须在执行任何 `gh issue comment` 或 `gh issue close` **之前**展示
- Jaccard 本地计算——无外部 API，无库，纯粹基于已获取数据的集合运算
- 所有评论的署名：`*Triaged via Claude Code /issue-triage*`

---

## 相关技能：/pr-triage

| | `/issue-triage` | `/pr-triage` |
|--|----------------|--------------|
| **范围** | Issue 积压 | PR 积压 |
| **适用场景** | 跟进报告者反馈、定期清理 issue | 跟进 PR 积压 |
| **阶段** | 3 个（审计 + 深度分析 + 操作） | 3 个（审计 + 深度审查 + 评论） |
| **智能体** | 每个 issue 并行子智能体 | 每个 PR 并行子智能体 |
| **重复检测** | 标题+正文的 Jaccard 相似度 | PR 间文件重叠百分比 |
| **操作** | 评论 / 标签 / 关闭 | GitHub 审查评论 |
| **验证** | 执行前通过 AskUserQuestion 确认 | 发布前通过 AskUserQuestion 确认 |

**决策规则**：`/issue-triage` 用于 issue 积压管理，`/pr-triage` 用于代码审查积压管理。
