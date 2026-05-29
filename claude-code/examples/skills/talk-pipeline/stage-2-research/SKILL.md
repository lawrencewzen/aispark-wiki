> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: talk-stage2-research
description: "执行 git 考古、变更日志分析，并通过交叉比对 git 历史与原始材料构建经过验证的事实时间线。仅限 REX 模式 —— 在 Concept 模式下自动跳过。当构建 REX 演讲且需要从 git 仓库获取已验证的提交指标、发布时间线和贡献者数据时使用。"
tags: [talk, pipeline, presentation, stage-2, git]
allowed-tools: "Write, Read, Bash"
effort: low
---

# 演讲第 2 阶段：调研（仅限 REX 模式）

为 REX 演讲构建 git 证明材料。交叉比对 git 历史、CHANGELOG 和第 1 阶段摘要，生成经过验证的时间线和速度分析。

**在 `--concept` 模式下自动跳过** —— 仅当原始材料是具有 git 仓库访问权限的 REX 时运行。

## 何时使用本技能

- 构建 REX 演讲时，完成第 1 阶段（提取）之后
- 拥有项目 git 仓库访问权限时
- 需要将原始材料中提到的指标与实际 git 数据进行核实时

## 本技能做什么

1. **读取摘要** —— 从第 1 阶段了解时间段和主题
2. **git 考古** —— 提取速度指标（仅只读命令）
3. **变更日志分析** —— 扫描版本发布、功能和已记录指标
4. **交叉比对** —— 对齐 git、CHANGELOG 和摘要
5. **构建时间线** —— 经过验证的日期，而非估算
6. **写入 3 个输出文件**

## 输入

- `talks/{YYYY}-{slug}-summary.md`（来自第 1 阶段 —— 必需）
- `repo_path` —— git 仓库的绝对路径
- 可选：若 CHANGELOG 路径与 `{repo_path}/CHANGELOG.md` 不同，则提供

## 输出

三个文件：
- `talks/{YYYY}-{slug}-git-archaeology.md`
- `talks/{YYYY}-{slug}-changelog-analysis.md`
- `talks/{YYYY}-{slug}-timeline.md`

## Git 命令（仅只读）

```bash
# 按月统计提交数
git -C {repo_path} log --pretty=format:"%Y-%m" | sort | uniq -c

# 按贡献者统计提交数
git -C {repo_path} shortlog -sn --no-merges

# 首次和最后日期
git -C {repo_path} log --pretty=format:"%ad" --date=short | tail -1
git -C {repo_path} log --pretty=format:"%ad" --date=short | head -1

# 合并的 PR 数（若遵循合并提交规范）
git -C {repo_path} log --merges --oneline | wc -l

# 标签（发布版本）
git -C {repo_path} tag --sort=version:refname

# 速度峰值（最繁忙的月份）
git -C {repo_path} log --pretty=format:"%Y-%m" | sort | uniq -c | sort -rn | head -5
```

## 输出格式

### git-archaeology.md

```markdown
# Git 考古 — {slug}

**仓库**: {repo_path}
**分析时间段**: {start date} → {end date}
**执行命令**: {list}

## 全局指标

| 指标 | 值 | 来源 |
|--------|-------|--------|
| 总提交数 | {n} | git log |
| 总合并/PR 数 | ~{n} | git log --merges |
| 总发布数（标签） | {n} | git tag |
| 人工贡献者数 | {n} | git shortlog |
| 覆盖时间段 | {n} 个月 | 首次 → 最后日期 |

## 月度速度

| 月份 | 提交数 | 备注 |
|-------|---------|-------|
| {YYYY-MM} | {n} | {如有特殊情况则说明} |
...

## 贡献者

| 排名 | 姓名 | 提交数 | 占比 |
|------|------|---------|---|
| 1 | {name} | {n} | {%} |
...

---
*由 talk-stage2-research 生成 — {date}*
```

### changelog-analysis.md

```markdown
# 变更日志分析 — {slug}

**来源**: {CHANGELOG 路径}
**分析版本数**: {n}（v{first} → v{last}）

## 各版本功能（摘要）

| 版本 | 日期 | 主要功能 | 提及的指标 |
|---------|------|--------------|------------------|
| {version} | {date} | {features} | {metrics} |
...

## 发现的规律

### 加速期
{高速度时期及其驱动因素}

### 转折点
{转向、方向变更、重大发布}

### CHANGELOG 中可验证的指标
{CHANGELOG 中提到的所有数字及其对应版本的详尽列表}

---
*由 talk-stage2-research 生成 — {date}*
```

### timeline.md

```markdown
# 事实时间线 — {slug}

**时间段**: {start} → {end}（{n} 个月）
**交叉比对来源**: 摘要 × Git 历史 × CHANGELOG

---

## 逐月时间线

| 月份 | 提交数 | 发布数 | 版本 | 主要功能 | 演讲事件 |
|-------|---------|----------|----------|--------------|-----------|
| {YYYY-MM} | {n} | {n} | {versions} | {features} | {轶事/事件} |
...

## 交叉比对：冲突与不一致

{若各来源之间的日期或指标存在矛盾，在此记录}

---
*由 talk-stage2-research 生成 — {date}*
*来源: git log × {CHANGELOG 路径} × {摘要路径}*
```

## 关键规则

- **仅只读** —— 不执行任何会修改仓库状态的 git 命令
- **先验证再断言** —— git 中找不到的日期 = "未经验证"
- **交叉比对** —— 标记来源间的不一致，不要随意选择其中一个
- **粒度** —— 最低按月；若时间段短（< 2 个月）则按周

## 反模式

- 执行会修改仓库的 git 命令
- 估算日期而非在 git 中核实
- 对指标取近似值而不作说明
- 静默合并来自不同来源的矛盾数据
- 遗漏安静期 —— 平台期同样说明问题

## 验证清单

- [ ] 仅执行了只读 git 命令
- [ ] 时间线覆盖摘要中的完整时间段
- [ ] 所有指标均有明确来源（git 或 CHANGELOG）
- [ ] 来源间的冲突已在专门章节中标记
- [ ] 3 个文件已生成并保存

## 提示

- 时间线是第 3 阶段（概念）用于丰富概念评分的素材 —— 高质量的时间线能让概念目录更强
- git 速度的"安静期"往往对应最艰难的工程挑战 —— 值得记录
- 若 CHANGELOG 没有版本标签，改用基于日期的锚定方式

## 相关

- [第 1 阶段：提取](../stage-1-extract/SKILL.md) —— 前置技能
- [第 3 阶段：概念](../stage-3-concepts/SKILL.md) —— 读取本阶段时间线
- [编排器](../orchestrator/SKILL.md)
