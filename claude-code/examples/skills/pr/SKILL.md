> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: pr
description: 分析变更、检测范围问题，并创建结构良好的 PR
argument-hint: "[--base <branch>] [--draft]"
effort: medium
disable-model-invocation: true
---

# 创建 Pull Request

分析变更、检测范围问题，并按项目规范创建结构良好的 PR。

## 流程

1. **分析变更**：根据文件、提交和目录计算复杂度评分
2. **检测范围问题**：若 PR 过大或混合了不相关的变更，则发出警告
3. **建议拆分**：如有必要，按范围对提交分组并提议分拆为独立 PR
4. **收集信息**：询问类型、目标分支、草稿状态、标签
5. **生成内容**：创建 TLDR + 描述 + 检查清单
6. **创建 PR**：以正确格式执行 `gh pr create`
7. **提醒后续步骤**：显示 PR 创建后的检查清单（SonarQube、Claude Review）

## 复杂度评分

计算 PR 复杂度，判断是否需要拆分：

| 标准 | 权重 | 说明 |
|-----------|--------|-------------|
| 代码文件 | x2 | `*.ts, *.tsx`（不含测试） |
| 测试文件 | x0.5 | `*.test.ts, *.spec.ts` |
| 配置文件 | x1 | `*.json, *.yml, *.md` |
| 目录数 | x3 | 不同的 `src/*` 目录 |
| 提交数 | x1 | 提交数量 |

**阈值**：0-15 ✅ 正常 | 16-25 ⚠️ 偏大 | 26+ 🔴 建议拆分

## 范围一致性

| 模式 | 结论 |
|---------|---------|
| 单一范围 | ✅ OK |
| 相关范围（sessions + calendar） | ✅ OK |
| 不相关范围（payments + auth） | 🔴 拆分 |
| feat + fix 同一范围 | ✅ OK |
| feat + fix 不同范围 | 🔴 拆分 |

## 拆分建议格式

建议拆分时，显示：

```
🔴 Scope trop large (score: 32)

Commits par scope :
├── payments (5 commits, 8 fichiers)
│   ├── feat(payments): add Stripe checkout
│   └── fix(payments): handle currency
│
└── notifications (3 commits, 6 fichiers)
    └── feat(notifications): add email templates

💡 Suggestion :
1. PR #1 : feature/payments-stripe → Commits payments
2. PR #2 : feature/notifications → Commits notifications

Options :
[A] Continuer avec une seule PR (non recommandé)
[B] Découper (semi-auto - commandes git fournies)
[C] Voir détail fichiers
```

**半自动拆分**提供可直接复制粘贴的命令：
```bash
git checkout develop
git checkout -b feature/payments-stripe
git cherry-pick abc1234 def5678
git push -u origin feature/payments-stripe
```

## 需要询问的问题

1. **类型**：feature | fix | tech | docs | security
2. **目标分支**：显示最近的分支（develop、main、其他）
3. **草稿**：是（进行中）| 否（可供审查）
4. **标签**：基于类型 + 可选项（breaking-change、security）

## PR 标题格式

```
<type>(<scope>): <description>
```

示例：
- `feat(payments): add Stripe checkout integration`
- `fix(sessions): resolve timezone calculation bug`

## PR 正文模板

```markdown
## TLDR
<!-- 最多 2 行 - 执行摘要 -->

---

## 类型
{Feature | Fix | Tech | Docs | Security}

## 描述
{背景与变更说明}

## 技术变更
{主要修改列表}

## 测试
- [ ] 单元测试已添加/通过
- [ ] 手动测试已完成

## 检查清单
- [ ] 代码符合规范
- [ ] 无遗留 console.log
- [ ] 类型检查通过（`pnpm typecheck`）

---

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## 可用标签

| 标签 | 颜色 | 使用场景 |
|-------|-------|----------|
| `feature` | 🟢 | 新功能 |
| `fix` | 🔴 | 缺陷修复 |
| `tech` | 🔵 | 重构、技术债务 |
| `docs` | 📘 | 仅文档变更 |
| `security` | 🟣 | 安全修复 |
| `breaking-change` | ⚫ | 破坏性变更 |
| `WIP` | 🟡 | 进行中（草稿） |

## 执行命令

```bash
# 1. 获取基础分支（通常为 develop）
BASE_BRANCH="develop"

# 2. 计算复杂度评分
CODE=$(git diff --name-only $BASE_BRANCH..HEAD | grep -E '\.(ts|tsx)$' | grep -v test | wc -l)
TESTS=$(git diff --name-only $BASE_BRANCH..HEAD | grep -E '\.test\.|\.spec\.' | wc -l)
DIRS=$(git diff --name-only $BASE_BRANCH..HEAD | cut -d'/' -f1-2 | sort -u | wc -l)
COMMITS=$(git rev-list --count $BASE_BRANCH..HEAD)
SCORE=$((CODE * 2 + TESTS / 2 + DIRS * 3 + COMMITS))

# 3. 从提交中获取范围
git log --oneline $BASE_BRANCH..HEAD --format="%s" | sed -n 's/^\w*(\([^)]*\)).*/\1/p' | sort | uniq -c

# 4. 用于选择的最近分支
git branch --sort=-committerdate --format='%(refname:short)' | head -5

# 5. 创建 PR
gh pr create \
  --title "<type>(<scope>): <description>" \
  --body "$BODY" \
  --base $BASE_BRANCH \
  --label "<label>" \
  --draft  # 如果是进行中状态
```

## PR 创建后的输出

PR 创建后，必须显示：

```
✅ PR créée : https://github.com/org/repo/pull/XXX

📋 Prochaines étapes automatiques :
   • SonarQube analysera la qualité du code (bugs, vulnérabilités, code smells)
   • Claude Code Review fournira un feedback IA sur votre PR

⏳ Pensez à surveiller ces analyses dans les prochaines minutes.
   Si des problèmes sont détectés, corrigez-les avant de demander une review humaine.
```

## 边界情况

| 场景 | 处理方式 |
|-----------|----------|
| 提交中无范围信息 | 按目录分析 |
| 非规范提交 | 警告 + 手动询问类型 |
| 无提交（与基础分支相同） | 报错："Aucun changement" |
| 仅一个提交 | 使用提交信息作为标题 |
| 合并提交 | 忽略（`--no-merges`） |

## 用法

```
/pr
/pr --base main
/pr --draft
```

目标：$ARGUMENTS（可选：--base、--draft）
