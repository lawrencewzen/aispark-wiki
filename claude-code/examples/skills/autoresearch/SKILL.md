> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: autoresearch
description: 自主改进循环——扫描代码库指标，生成实验文件骨架，运行智能体驱动的迭代直到指标改善
argument-hint: "[--scaffold <loop-name>] [--run <loop-name>] [--status]"
effort: high
disable-model-invocation: true
---

# Autoresearch — 自主改进循环

扫描代码库质量指标，提议改进循环，并运行自主智能体迭代。灵感来源于 [karpathy/autoresearch](https://github.com/karpathy/autoresearch)——从机器学习研究迁移到代码质量领域。

**核心概念**：智能体提议一项代码变更，运行度量，若指标改善则保留变更，否则通过 `git reset` 回滚，如此循环直到手动停止。

**时间估计**：扫描约 30 秒 | 每次迭代：取决于范围 | 循环：无限运行直到你停止

---

## 模式 1：扫描（默认）

测量当前状态，检测已有循环，提议下一步操作。

### 指令

运行以下指标并展示优先级排序的提案表格。

**第 1 步：测量代码库指标**

根据你的项目规范调整 grep 模式。以下为 TypeScript 默认值——请按你的技术栈调整。

```bash
# M1：函数声明（推荐箭头函数）
M1=$(grep -r "export function " src/ --include="*.ts" --include="*.tsx" -l 2>/dev/null | wc -l | tr -d ' ')

# M2：接口声明（推荐类型别名）
M2=$(grep -r "export interface " src/ --include="*.ts" --include="*.tsx" -l 2>/dev/null | wc -l | tr -d ' ')

# M3：ESLint 禁用注释
M3=$(grep -r "eslint-disable" src/ --include="*.ts" --include="*.tsx" -l 2>/dev/null | wc -l | tr -d ' ')

# M4：类型转换为 any
M4=$(grep -r " as any" src/ --include="*.ts" --include="*.tsx" -l 2>/dev/null | wc -l | tr -d ' ')

# M5：TODO 注释
M5=$(grep -r "// TODO" src/ --include="*.ts" --include="*.tsx" -l 2>/dev/null | wc -l | tr -d ' ')
```

**第 2 步：检测已有循环**

```bash
for dir in scripts/autoresearch/loop-*/; do
  [ -d "$dir" ] || continue
  LOOP_NAME=$(basename "$dir")
  # Check if loop has results
  if [[ -f "$dir/results.tsv" ]]; then
    ITERS=$(wc -l < "$dir/results.tsv" | tr -d ' ')
    BEST=$(sort -t$'\t' -k2 -n "$dir/results.tsv" | head -1 | cut -f2)
    echo "ACTIVE:$LOOP_NAME:iterations=$ITERS:best=$BEST"
  else
    echo "SCAFFOLDED:$LOOP_NAME"
  fi
done
```

**第 3 步：展示结果**

```
Autoresearch 扫描 — {日期}

代码库指标：

| # | 循环              | 指标              | 当前值  | 目标值 | 优先级 | 风险 |
|---|-------------------|-------------------|---------|--------|--------|------|
| A | loop-remove-as-any| `as any` 类型转换 | {M4}    | 0      | P1     | 低   |
| B | loop-eslint-disable| eslint-disable   | {M3}    | 0      | P2     | 中   |
| C | loop-export-fn    | export function   | {M1}    | 0      | P1     | 低   |
| D | loop-interface-type| export interface | {M2}    | 0      | P1     | 低   |
| E | loop-todo-comments| TODO 注释         | {M5}    | 0      | P3     | 低   |

已有循环：{检测到的循环，或"暂无"}

推荐下一步（P1，低风险）：
  /autoresearch --scaffold loop-remove-as-any
  然后编写 program.md，创建 worktree，并运行循环。
```

---

## 模式 2：`--scaffold <loop-name>`

为一个循环生成 3 个机械文件。**不生成 `program.md`**——需自行编写，以编码项目特定约束。

### 指令

在 `scripts/autoresearch/{loop-name}/` 下创建以下文件：

**`measure.sh`** — 评估测量脚本（单一指标，返回整数）：

```bash
#!/usr/bin/env bash
# measure.sh — {loop-name}
# 返回一个整数。方向：越低越好（除非循环目标是覆盖率/得分）。
set -euo pipefail
grep -r "PATTERN" src/ --include="*.ts" --include="*.tsx" 2>/dev/null | wc -l | tr -d ' '
```

**`direction.txt`** — 改进方向：

```
lower
```

（若指标为测试覆盖率或质量评分，使用 `higher`。）

**`files.txt`** — 智能体应操作的范围：

```
src/
```

创建文件后，展示：

```
循环已生成骨架：scripts/autoresearch/{loop-name}/

  measure.sh  : {pattern} 在 {scope} 中 → 当前 {N} 处
  direction   : lower（越少越好）
  files.txt   : src/

当前指标：{N}（目标：0）

后续步骤：
  1. 编写 program.md——智能体行为、约束、可操作/不可操作的内容
     参考：scripts/autoresearch/loop-remove-as-any/program.md
  2. 创建 worktree：/worktree feature/autoresearch-{loop-name}
  3. cd 进入该 worktree
  4. bash scripts/autoresearch/runner.sh {loop-name} 0 15
```

---

## 模式 3：`--run <loop-name>`

执行自主循环。智能体无限运行——满意后手动停止。

### 指令

**验证前置条件：**

```bash
[ -f "scripts/autoresearch/{loop-name}/measure.sh" ] || { echo "ERROR: measure.sh 缺失。请先运行 --scaffold。"; exit 1; }
[ -f "scripts/autoresearch/{loop-name}/program.md" ] || { echo "ERROR: program.md 缺失。请先编写——这里编码了你的约束条件。"; exit 1; }
```

**运行循环：**

开始前完整阅读 `scripts/autoresearch/{loop-name}/program.md`。然后进入以下循环——重复直到停止：

```
循环迭代 #{N}

1. 当前指标：bash scripts/autoresearch/{loop-name}/measure.sh
2. 读取 program.md 约束
3. 针对 files.txt 中的文件提议一项定向变更
4. 应用变更
5. 重新测量：bash scripts/autoresearch/{loop-name}/measure.sh
6. 评估：
   - direction=lower 且新值 < 旧值 → 保留（git add -p && git commit -m "autoresearch: {描述}"）
   - 否则 → 回滚（git checkout -- .）
7. 记录到 results.tsv：{时间戳}\t{指标}\t{状态}\t{描述}
8. 继续迭代 #{N+1}
```

**停止条件**（来自 program.md）：
- 指标达到目标（例如 0）
- 没有更多可机械完成的变更
- 用户手动停止

**每次迭代展示：**

```
[迭代 #{N}] 指标：{前} → {后} | {已保留/已回滚} | {变更描述}
```

---

## 模式 4：`--status`

展示项目中所有循环的状态。

### 指令

```bash
for dir in scripts/autoresearch/loop-*/; do
  [ -d "$dir" ] || continue
  NAME=$(basename "$dir")
  CURRENT=$(bash "$dir/measure.sh" 2>/dev/null || echo "?")
  ITERS=$([ -f "$dir/results.tsv" ] && wc -l < "$dir/results.tsv" | tr -d ' ' || echo "0")
  KEPT=$([ -f "$dir/results.tsv" ] && grep -c "KEPT" "$dir/results.tsv" || echo "0")
  echo "$NAME | current: $CURRENT | iters: $ITERS | kept: $KEPT"
done
```

展示：

```
Autoresearch 状态

| 循环                | 当前值  | 迭代次数   | 已保留 | 状态      |
|---------------------|---------|------------|--------|-----------|
| loop-remove-as-any  | {N}     | {N}        | {N}    | 运行中    |
| loop-export-fn      | {N}     | 0          | 0      | 已生成骨架|
```

---

## 编写 `program.md` — 最重要的文件

`program.md` 是智能体的行为契约。需自行编写——永远不要自动生成。它必须编码智能体在你的特定代码库中可以/不可以操作的内容。

**最小结构：**

```markdown
# Program: {loop-name}

## 目标
将 `src/` 中的 `{指标}` 减少到 0。每次迭代只做一项机械变更。

## 测量
bash scripts/autoresearch/{loop-name}/measure.sh
越低越好。目标：0。

## 可以做的事
- 将 `export function X(` 替换为 `export const X = (`
- 保持函数签名完全不变

## 不可以做的事
- 修改测试文件
- 更改函数签名
- 操作 src/ 以外的文件
- 每次迭代做多项变更

## 停止条件
- 指标 = 0
- 没有更多可机械替换的内容
```

---

## 模式背景

本命令实现了来自 [karpathy/autoresearch](https://github.com/karpathy/autoresearch) 的 **autoresearch 循环**模式：

| 机器学习研究（karpathy）| 代码质量（本命令）|
|------------------------|-------------------|
| 修改 `train.py` | 修改 `src/` 文件 |
| 测量 `val_bpb` | 测量 grep 计数 |
| 5 分钟 GPU 预算 | 每次迭代一项原子变更 |
| val_bpb 改善则保留 | 计数下降则保留 |
| 否则 `git reset` | 否则 `git checkout -- .` |
| `program.md` = 智能体技能 | `program.md` = 智能体技能 |

核心洞见：固定的客观指标 + git 作为回滚机制 = 安全的自主迭代。智能体无需每次变更都获得人工审批，因为所有不良变更都会被自动回滚。

---

## 使用方法

**扫描并提议循环：**
```
/autoresearch
```

**为特定循环生成骨架文件：**
```
/autoresearch --scaffold loop-remove-as-any
```

**运行自主循环（编写 program.md 后）：**
```
/autoresearch --run loop-remove-as-any
```

**查看所有循环状态：**
```
/autoresearch --status
```

$ARGUMENTS
