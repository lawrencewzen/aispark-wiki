> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: pr-triage
description: "4 阶段拉取请求待办管理：审计、深度代码审查、已验证评论发布、可选工作树设置。适用于对拉取请求进行分类处理、跟进待处理的代码审查、或管理积压的开放 PR。参数：'all' 审查全部，指定 PR 编号聚焦处理（如 '42 57'），'en'/'fr' 指定语言，无参数 = 仅审计。"
allowed-tools: Bash Read
effort: medium
---

# PR 分类处理

面向维护者的 4 阶段工作流：自动审计所有开放 PR、按需深度审查（通过并行 Agent）、已验证评论发布、以及可选的本地审查工作树设置。

## 何时使用本技能

| 技能 | 用途 | 输出 |
|-------|-------|--------|
| `/pr-triage` | 对 PR 待办进行分类、审查并发表评论 | 分类处理表格 + 审查报告 + 已发布评论 |
| `/review-pr` | 深度审查单个 PR | 行内 PR 审查 |

**触发条件**：
- 手动：`/pr-triage` 或 `/pr-triage all` 或 `/pr-triage 42 57`
- 自动触发：当超过 5 个 PR 无人审查，或检测到超过 14 天未活动的陈旧 PR 时

---

## 语言

- 检查传入技能的参数
- 若为 `en` 或 `english` → 表格和摘要使用英文
- 若为 `fr`、`french` 或无参数 → 使用法语（默认）
- 注意：GitHub 评论（第 3 阶段）始终使用英文（面向国际受众）

---

## 配置

工作流中使用的阈值，可根据项目需要编辑：

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `staleness_days` | 14 | 标记为陈旧之前的无活动天数 |
| `overlap_threshold` | 50% | 触发重叠标记的共享文件比例 |
| `cluster_min_prs` | 3 | 触发集群建议的单作者 PR 数量 |
| `xl_cutoff_additions` | 1000 | 超过此新增行数则分类为 XL |
| `xl_cutoff_files` | 10 | 超过此修改文件数则认为 PR "过大" |

---

## 前提条件

```bash
git rev-parse --is-inside-work-tree
gh auth status
```

若任一命令失败，停止执行并说明缺少什么。

---

## 第 1 阶段 — 审计（始终执行）

### 数据收集（并行命令）

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
gh pr list --state open --limit 50 \
  --json number,title,author,createdAt,updatedAt,additions,deletions,changedFiles,isDraft,mergeable,reviewDecision,statusCheckRollup,body
gh api "repos/{owner}/{repo}/collaborators" --jq '.[].login'
```

**协作者回退方案**：若 `gh api .../collaborators` 返回 403/404：
```bash
gh pr list --state merged --limit 10 --json author --jq '.[].author.login' | sort -u
```
若仍不明确，通过 `AskUserQuestion` 询问用户。

对每个 PR，获取审查信息和修改文件列表：

```bash
gh api "repos/{owner}/{repo}/pulls/{num}/reviews" \
  --jq '[.[] | .user.login + ":" + .state] | join(", ")'
gh pr view {num} --json files --jq '[.files[].path] | join(",")'
```

**注意**：获取文件列表每个 PR 需要 1 次 API 调用——对于 20 个以上的 PR，优先处理重叠候选项。`author` 字段是对象，始终提取 `.author.login`。

### 分析

**规模分类**：
| 标签 | 新增行数 |
|-------|-----------|
| XS | < 50 |
| S | 50–200 |
| M | 200–500 |
| L | 500–1000 |
| XL | > 1000 |

规模格式：`+{additions}/-{deletions}, {files} files ({label})`

**检测项目**：
- **重叠**：比较各 PR 的文件列表——若共享文件超过 50% → 进行交叉引用
- **集群**：某作者有 3 个以上开放 PR → 建议审查顺序（从最小的开始）
- **陈旧**：超过 14 天无活动 → 标记为"陈旧"
- **CI 状态**：通过 `statusCheckRollup` 获取 → `clean`（通过）/ `unstable`（不稳定）/ `dirty`（失败）
- **审查状态**：已批准 / 需要更改 / 无审查

**PR ↔ 问题关联**：
- 扫描每个 PR 的 `body`，查找 `fixes #N`、`closes #N`、`resolves #N`（不区分大小写）
- 若找到，在表格的操作/状态列显示：`Fixes #42`

**分类规则**：

_内部 PR_：作者在协作者列表中

_外部 — 就绪_：新增行数 ≤ 1000 且 修改文件 ≤ 10 且 `mergeable` ≠ `CONFLICTING` 且 CI 通过或不稳定

_外部 — 有问题_：满足以下任一条件：
- 新增行数 > 1000 或 修改文件 > 10
- 或 `mergeable` == `CONFLICTING`（存在合并冲突）
- 或 CI 失败（`statusCheckRollup` 包含失败项）
- 或与另一个开放 PR 存在重叠（共享文件超过 50%）

### 输出 — 分类处理表格

```
## 开放 PR ({count})

### 内部 PR
| PR | 标题 | 规模 | CI | 状态 |
| -- | ----- | ---- | -- | ------ |

### 外部 — 待审查
| PR | 作者 | 标题 | 规模 | CI | 审查 | 操作 |
| -- | ------ | ----- | ---- | -- | ------- | ------ |

### 外部 — 有问题
| PR | 作者 | 标题 | 规模 | 问题 | 建议操作 |
| -- | ------ | ----- | ---- | ------- | ------------------ |

### 摘要
- 快速合并项：{可快速合并的 XS/S PR}
- 风险项：{重叠、XL 规模、CI 失败}
- 集群：{拥有 3 个以上 PR 的作者}
- 陈旧项：{超过 14 天无活动的 PR}
- 重叠项：{涉及相同文件的 PR}
```

0 个 PR → 显示 `No open PRs.` 并停止。

### 第 1 阶段结束后的导航

显示分类处理表格后，通过 `AskUserQuestion` 询问：

```
question: "下一步您想做什么？"
header: "下一步"
options:
  - label: "第 2 阶段 — 深度审查"
    description: "使用代码审查 Agent 分析所选 PR 并生成评论草稿"
  - label: "第 4 阶段 — 创建工作树"
    description: "设置本地工作树以进行实际代码审查（跳过评论生成）"
  - label: "完成"
    description: "在此结束工作流"
```

注意：第 3 阶段（发布评论）不在此处提供——它需要第 2 阶段生成的草稿。若用户选择"第 4 阶段"，第 2 → 第 3 阶段仍可在之后访问。

### 自动复制

显示分类处理表格后，使用平台对应命令复制到剪贴板：

```bash
UNAME=$(uname -s)
if [ "$UNAME" = "Darwin" ]; then
  pbcopy <<'EOF'
{full triage table}
EOF
elif command -v xclip &>/dev/null; then
  echo "{full triage table}" | xclip -selection clipboard
elif command -v wl-copy &>/dev/null; then
  echo "{full triage table}" | wl-copy
elif command -v clip.exe &>/dev/null; then
  echo "{full triage table}" | clip.exe
fi
```

确认提示：`Triage table copied to clipboard.`（英文）/ `Tableau copié dans le presse-papier.`（法文）

---

## 第 2 阶段 — 深度审查（按需）

### PR 选择

**若传入参数**：
- `"all"` → 所有外部 PR
- 编号（`"42 57"`）→ 仅处理这些 PR
- 无参数 → 通过 `AskUserQuestion` 提议

**若无参数**，显示：

```
question: "您想深度审查哪些 PR？"
header: "深度审查"
multiSelect: true
options:
  - label: "全部外部"
    description: "使用并行代码审查 Agent 审查 {N} 个外部 PR"
  - label: "仅有问题的"
    description: "聚焦 {M} 个风险 PR（CI 失败、规模过大、存在重叠）"
  - label: "仅就绪的"
    description: "审查 {K} 个可合并的 PR"
  - label: "跳过"
    description: "在此停止——仅审计"
```

**草稿 PR 的处理方式**：
- 草稿 PR 从"全部外部"和"仅就绪的"中排除
- 草稿 PR 包含在"仅有问题的"中（需要关注）
- 若要审查草稿：明确输入其编号（如 `42`）

若选择"跳过"→ 结束工作流。

### 执行审查

对每个选定的 PR，通过 **Task 工具并行**启动 `code-reviewer` Agent：

```
subagent_type: code-reviewer
model: sonnet
prompt: |
  Review PR #{num}: "{title}" by @{author}

  **Metadata**: +{additions}/-{deletions}, {changedFiles} files ({size_label})
  **CI**: {ci_status} | **Reviews**: {existing_reviews} | **Draft**: {isDraft}

  **PR Body**:
  {body}

  **Diff**:
  {gh pr diff {num} output}

  Apply your security and architecture expertise. Use the project-specific checklist
  from the SKILL.md Configuration section if available.

  Return structured review:
  ### Critical Issues
  ### Important Issues
  ### Suggestions
  ### What's Good

  Be specific: quote file:line, explain the issue, suggest the fix.
```

**并行 Agent 不可用时的回退方案**：逐个顺序审查 PR。通知用户：`Running sequential review (parallel agents not available).`

通过以下命令获取差异：
```bash
gh pr diff {num}
gh pr view {num} --json body,title,author -q '{body: .body, title: .title, author: .author.login}'
```

汇总所有报告，所有审查完成后显示摘要。

---

## 第 3 阶段 — 评论（必须验证）

### 草稿生成

对每个已审查的 PR，使用模板 `templates/review-comment.md` 生成 GitHub 评论。

**规则**：
- 语言：**英文**（面向国际受众）
- 语气：专业、建设性、客观
- 至少包含 1 个正面评价
- 在适当时引用代码行（格式 `file:42`）

### 显示与验证

**显示全部评论草稿**，格式如下：

```
---
### 草稿 — PR #{num}: {title}

{full comment}

---
```

然后通过 `AskUserQuestion` 请求验证：

```
question: "这些评论已就绪，您想发布哪些？"
header: "发布评论"
multiSelect: true
options:
  - label: "全部（{N} 条评论）"
    description: "发布到所有已审查的 PR"
  - label: "PR #{x} — {title_truncated}"
    description: "仅发布到此 PR"
  - label: "无"
    description: "取消——不发布任何内容"
```

（每个 PR 生成一个选项 + "全部" + "无"）

### 发布

对每条已验证的评论：

```bash
gh pr comment {num} --body-file - <<'REVIEW_EOF'
{comment}
REVIEW_EOF
```

逐条确认：`Comment posted on PR #{num}: {title}`

若选择"无"→ `No comments posted. Workflow complete.`

---

## 项目专项检查清单

在第 2 阶段将您技术栈的检查清单添加到 Agent 提示中。各技术栈示例：

**Node.js / TypeScript**：
- 使用 `any` 类型需有明确理由
- `async/await` 错误处理（try/catch 或 `.catch()`）
- 无未处理的 Promise 拒绝
- 在 API 边界进行输入验证

**Python**：
- 所有公共函数添加类型注解
- 异常要具体（不使用裸 `except:`）
- 资源清理（使用 `with` 语句、上下文管理器）
- 无可变默认参数

**Rust**：
- 用 `Result<T, E>` 配合 `.context()` 传递错误链（生产代码中不使用 `.unwrap()`）
- 热路径上不无故使用 `clone()`
- 静态正则使用 `lazy_static!` 或 `once_cell`
- 所有权不明显时添加生命周期注解

**Go**：
- 显式错误处理（不使用 `_` 丢弃，除非有注释说明）
- 使用 `defer` 进行资源清理
- 并发代码中传递 Context
- 无 goroutine 泄漏

**通用**（与技术栈无关）：
- 无密钥或硬编码凭证
- 新的公共函数有测试覆盖
- 破坏性变更在 PR 描述中记录
- 新增依赖有明确的理由说明

---

---

## 第 4 阶段 — 工作树设置（按需）

为每个选定的 PR 创建本地 git 工作树，使您可以在不切换分支的情况下运行、测试或审查代码。

**从不自动触发**——仅通过第 1 阶段导航或用户明确请求触发。

### 步骤 4.1 — 缓存检查 + PR 列表

**缓存检查**：在使用第 1 阶段的数据之前，验证其是否在 30 分钟以内：

```bash
CACHE_FILE="/tmp/pr-triage-prs.json"
CACHE_AGE=$(( $(date +%s) - $(stat -f %m "$CACHE_FILE" 2>/dev/null || echo 0) ))
if [ "$CACHE_AGE" -gt 1800 ]; then
  echo "STALE_CACHE"
fi
```

若显示 `STALE_CACHE` → 在继续之前重新执行第 1 阶段数据收集。

**过滤**：排除草稿 PR 和机器人 PR（Dependabot、renovate 等）：

```bash
python3 -c "
import json
prs = json.load(open('/tmp/pr-triage-prs.json'))
filtered = [
  p for p in prs
  if not p['isDraft']
  and not any(bot in p['author']['login'].lower() for bot in ['dependabot', 'renovate', 'snyk'])
]
import sys; json.dump(filtered, sys.stdout, indent=2)
" > /tmp/pr-triage-phase4.json
```

过滤后 0 个 PR → 显示 `No reviewable PRs available for worktree (all are drafts or bots).` 并结束第 4 阶段。

**按作者分组显示**（有显示名称时使用，否则用登录名）：

```
## 可创建工作树的 PR（非草稿）

### Alice Martin (@alice)
  [1] #123 — feat(auth): add OAuth2 support
      分支: feat/oauth2  |  规模: M  |  CI: clean

### Bob Chen (@bob)
  [2] #456 — fix(api): handle empty response
      分支: fix/empty-response  |  规模: S  |  CI: dirty ⚠️
```

### 步骤 4.2 — 选择

通过 `AskUserQuestion` 询问（多选）：

```
question: "您想为哪些 PR 创建工作树？"
header: "工作树设置"
multiSelect: true
options:
  - label: "全部"
    description: "为所有 {N} 个列出的 PR 创建工作树"
  - label: "[1] #{num} — {title} ({author})"
    description: "分支: {branch} | 规模: {size} | CI: {ci}"
  - label: "无"
    description: "取消——返回菜单"
```

若选择"无"→ 结束第 4 阶段。

### 步骤 4.3 — 顺序创建

**执行模型**：Claude 对每个 PR **运行一个 bash 命令**，读取其输出，更新内部状态（已创建 / 已存在 / 失败），然后继续处理下一个。绝不使用循环包裹所有 PR。

对每个选定的 PR，Claude 明确设置变量后运行：

```bash
PR_NUM="123"
BRANCH_NAME="feat/oauth2"
WORKTREE_NAME="${BRANCH_NAME//\//-}"
REPO_ROOT="$(cd "$(git rev-parse --git-common-dir)/.." && pwd)"
WORKTREE_DIR="$REPO_ROOT/.worktrees/$WORKTREE_NAME"

# 是否已存在？
if [ -d "$WORKTREE_DIR" ]; then
  echo "STATUS:EXISTING:$PR_NUM:$WORKTREE_DIR"
  exit 0
fi

# .gitignore 检查（快速失败）
if ! grep -qE "^\.worktrees/?$" "$REPO_ROOT/.gitignore" 2>/dev/null; then
  echo "STATUS:GITIGNORE_MISSING:$PR_NUM"
  exit 1
fi

# 拉取远程分支
if ! git fetch origin "$BRANCH_NAME" 2>/tmp/wt-fetch-$PR_NUM.log; then
  echo "STATUS:FETCH_FAILED:$PR_NUM"
  exit 1
fi

mkdir -p "$REPO_ROOT/.worktrees"

# 创建工作树（无论本地分支是否存在）
if ! git branch --list "$BRANCH_NAME" | grep -q "$BRANCH_NAME"; then
  git worktree add -b "$BRANCH_NAME" "$WORKTREE_DIR" "origin/$BRANCH_NAME" \
    2>/tmp/wt-err-$PR_NUM.log
else
  git worktree add "$WORKTREE_DIR" "$BRANCH_NAME" \
    2>/tmp/wt-err-$PR_NUM.log
fi

if [ $? -ne 0 ]; then
  if grep -q "already checked out" /tmp/wt-err-$PR_NUM.log; then
    echo "STATUS:ALREADY_CHECKED_OUT:$PR_NUM"
  else
    echo "STATUS:CREATE_FAILED:$PR_NUM"
  fi
  exit 1
fi

# 可选：符号链接 node_modules（Node.js 项目——避免重新安装）
[ -d "$REPO_ROOT/node_modules" ] && ln -sf "$REPO_ROOT/node_modules" "$WORKTREE_DIR/node_modules"

# 复制 .worktreeinclude 中列出的项目专属文件（若存在）
if [ -f "$REPO_ROOT/.worktreeinclude" ]; then
  while IFS= read -r entry || [ -n "$entry" ]; do
    [[ "$entry" =~ ^#.*$ || -z "$entry" ]] && continue
    entry="$(echo "$entry" | xargs)"
    [ -e "$REPO_ROOT/$entry" ] && {
      mkdir -p "$(dirname "$WORKTREE_DIR/$entry")"
      cp -R "$REPO_ROOT/$entry" "$WORKTREE_DIR/$entry"
    }
  done < "$REPO_ROOT/.worktreeinclude"
fi

echo "STATUS:CREATED:$PR_NUM:$WORKTREE_DIR"
```

**状态处理**（Claude 在各 PR 之间维护内部状态）：

| 状态 | Claude 操作 |
|--------|--------------|
| `STATUS:CREATED:NUM:PATH` | 添加到"已创建"列表 |
| `STATUS:EXISTING:NUM:PATH` | 添加到"已存在"列表 → 在步骤 4.4 提供更新选项 |
| `STATUS:FETCH_FAILED:NUM` | 警告 + 继续处理下一个 PR |
| `STATUS:GITIGNORE_MISSING:NUM` | 快速失败：显示修复说明 + 停止第 4 阶段 |
| `STATUS:ALREADY_CHECKED_OUT:NUM` | 警告："该分支已在另一个工作树中检出，运行 `git worktree list` 查找。" |
| `STATUS:CREATE_FAILED:NUM` | 警告 + 继续处理下一个 PR |

**GITIGNORE_MISSING 修复说明**：
```
.worktrees/ 不在 .gitignore 中。请添加以避免意外提交工作树文件：
  echo ".worktrees/" >> .gitignore
然后重新运行第 4 阶段。
```

### 步骤 4.4 — 更新已存在的工作树

若收集到任何 `STATUS:EXISTING`，提供单一提示：

```
检测到已存在的工作树：
  PR #123 — .worktrees/feat-oauth2
  PR #789 — .worktrees/fix-session-leak

- [全部更新] 在所有已存在的工作树中执行 git pull --ff-only
- [#123] 仅更新 PR #123
- [跳过] 保持不变
```

对每个选择更新的工作树，Claude 运行（每个工作树一个命令）：

```bash
PR_NUM="123"
BRANCH_NAME="feat/oauth2"
WORKTREE_DIR="/abs/path/.worktrees/feat-oauth2"

cd "$WORKTREE_DIR" && git pull origin "$BRANCH_NAME" --ff-only 2>/tmp/wt-pull-$PR_NUM.log
echo "PULL_STATUS:$?:$PR_NUM"
```

若 `PULL_STATUS` ≠ 0：
```
⚠️ PR #123 — --ff-only 失败（分支已分叉）
   手动修复：cd .worktrees/feat-oauth2 && git pull --rebase
```

### 步骤 4.5 — 汇总

```
## 工作树已就绪

| PR | 作者 | 分支 | 路径 | 状态 |
|----|--------|--------|------|--------|
| #123 | Alice | feat/oauth2 | .worktrees/feat-oauth2 | 已创建 |
| #456 | Bob | fix/empty-response | .worktrees/fix-empty-response | 已创建 |
| #789 | Alice | fix/session-leak | .worktrees/fix-session-leak | 已更新（pull）|
| #321 | Carol | feat/chat | .worktrees/feat-chat | 拉取失败 ⚠️ |

注意：若某 PR 修改了 package.json，请手动安装依赖：
  cd .worktrees/<branch-name> && npm install   # 或 pnpm/yarn/bun

后续步骤：
  cd .worktrees/<branch-name>
  claude
```

### `.worktreeinclude` 约定

在仓库根目录创建 `.worktreeinclude` 文件，列出第 4 阶段复制到每个新工作树的文件。适用于未被 git 追踪的本地配置文件：

```
# .worktreeinclude
.env.local
.env.test
config/local.json
```

---

## 边界情况

| 情况 | 处理行为 |
|-----------|----------|
| 0 个开放 PR | 显示 `No open PRs.` + 停止 |
| 草稿 PR | 在表格中显示，除非明确选择否则跳过审查 |
| 未知 CI | CI 列显示 `?` |
| 审查 Agent 超时 | 显示部分错误，继续处理其他 PR |
| `gh pr diff` 为空 | 跳过此 PR，通知用户 |
| 超大 PR（> 5000 新增行数）| 警告："部分审查，差异已截断" |
| 协作者 API 403/404 | 回退至最近 10 个合并 PR 的作者 |
| 并行 Agent 不可用 | 顺序执行审查，通知用户 |
| 第 4 阶段：`.gitignore` 缺少 `.worktrees/` | 快速失败，显示修复说明，停止第 4 阶段 |
| 第 4 阶段：分支已被检出 | 显示 `git worktree list` 提示的警告，跳过此 PR |
| 第 4 阶段：缓存过期（> 30 分钟）| 在创建工作树前重新获取 PR 列表 |
| 第 4 阶段：PR 修改了 `package.json` | 在摘要中警告需手动运行安装 |
| 第 4 阶段：0 个非草稿 PR | 显示提示 + 结束第 4 阶段 |

---

## 注意事项

- 始终通过 `gh repo view` 获取 owner/repo，绝不硬编码
- 使用 `gh` CLI（而非 `curl` GitHub API），协作者列表除外
- `statusCheckRollup` 可能为 null → 视为 `?`
- `mergeable` 可以是 `MERGEABLE`、`CONFLICTING` 或 `UNKNOWN` → 将 `UNKNOWN` 视为 `?`
- 未经用户在对话中明确验证，绝不发布评论
- 评论草稿必须在任何 `gh pr comment` 执行前可见

---

## 关联：/review-pr

| | `/pr-triage` | `/review-pr` |
|--|-------------|--------------|
| **范围** | 完整 PR 待办 | 单个 PR |
| **适用场景** | 积压后跟进、周期性分类处理 | 审查特定的新 PR |
| **阶段** | 4 个（审计 + 深度审查 + 评论 + 工作树） | 1 个（仅审查） |
| **Agent** | 每个 PR 并行启动子 Agent | 单一会话 |
| **输出** | 分类处理表格 + 审查报告 + GitHub 评论 + 本地工作树 | 行内审查 |
| **验证** | 发布前通过 AskUserQuestion 确认 | 手动决定 |

**决策规则**：积压分类处理（5 个以上 PR）使用 `/pr-triage`，专注审查单个 PR 使用 `/review-pr`。若需在本地运行代码而非仅阅读差异，使用第 4 阶段。
