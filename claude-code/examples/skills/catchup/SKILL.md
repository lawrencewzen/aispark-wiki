> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: catchup
description: 在 /clear 后通过汇总近期工作与项目状态恢复上下文
argument-hint: "[branch] [--since <date>]"
effort: low
disable-model-invocation: true
---

# 上下文恢复（Context Catchup）

在执行 `/clear` 后恢复上下文——汇总近期工作内容与项目状态。

## 用途

使用 `/clear` 清除上下文后，用此命令快速重建以下内容的理解：
- 最近修改了哪些内容
- 当前项目状态
- 待处理的 TODO 和问题
- 从哪里继续工作

## 操作说明

### 步骤 1：Git 历史分析

```bash
# 最近提交（最近 10 条）
git log --oneline -10

# 最近 5 次提交涉及的文件
git diff --stat HEAD~5 2>/dev/null || git diff --stat $(git rev-list --max-parents=0 HEAD)

# 当前分支与状态
git branch --show-current
git status --short
```

### 步骤 2：近期变更汇总

```bash
# 今天的变更
git log --oneline --since="midnight" --author="$(git config user.name)" 2>/dev/null

# 未提交的工作
git diff --name-only
git diff --cached --name-only
```

### 步骤 3：TODO/FIXME 扫描

```bash
# 在最近修改的文件中查找未完成标记
git diff --name-only HEAD~5 2>/dev/null | head -20 | xargs grep -n "TODO\|FIXME\|XXX\|HACK" 2>/dev/null | head -30
```

### 步骤 4：项目状态检查

```bash
# 检查常见状态标志
[ -f "package.json" ] && echo "📦 Node project: $(jq -r '.name // "unnamed"' package.json)"
[ -f "Cargo.toml" ] && echo "🦀 Rust project: $(grep '^name' Cargo.toml | head -1)"
[ -f "pyproject.toml" ] && echo "🐍 Python project"
[ -f "go.mod" ] && echo "🐹 Go project: $(head -1 go.mod | cut -d' ' -f2)"

# 当前分支用途（从分支名推断）
BRANCH=$(git branch --show-current)
echo "🌿 Branch: $BRANCH"
```

## 输出格式

输出结构化汇总：

---

### 📍 上下文已恢复

**项目**：[来自 package.json/Cargo.toml 等的名称]
**分支**：[当前分支]
**最近活动**：[最后提交时间]

### 🔄 近期工作（最近 5 次提交）

1. [提交信息 1] - [涉及文件]
2. [提交信息 2] - [涉及文件]
...

### 📝 未提交的变更

- [已修改文件列表及变更简述]

### ⚠️ 待处理 TODO

- [文件:行号] TODO: [描述]
- [文件:行号] FIXME: [描述]

### 🎯 建议的后续步骤

基于近期活动：
1. [根据规律判断的最可能下一步]
2. [备选关注点]

---

## 使用示例

**长时间中断后恢复：**
```
/catchup
```
→ 完整上下文恢复

**快速状态检查：**
```
/catchup --brief
```
→ 仅查看提交记录和未提交变更

**聚焦特定领域：**
```
/catchup auth
```
→ 仅过滤与 auth 相关的变更

## 进阶技巧

1. **清除前先记录**：在提交信息或 CLAUDE.md 中写一条简要备注，再执行 `/clear`
2. **配合记忆库使用**：结合 `.claude/memory/` 文件实现持久化状态管理
3. **善用分支命名**：使用描述性分支名（如 `feat/user-auth`）有助于上下文恢复

$ARGUMENTS
