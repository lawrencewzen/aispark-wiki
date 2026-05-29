> 📚 **AI Spark Wiki** · Claude Code 知识库

# 会话复盘模板

在 Claude Code 会话结束时运行的结构化提示词。目标是趁知识还新鲜时将其沉淀下来，转化为持久规则——随着项目演进持续保持 `CLAUDE.md` 的准确性。

## 提示词

在会话结束时复制并粘贴给 Claude：

```
Session complete. Before we close, run a quick retrospective:

1. What patterns did we establish that should be permanent rules in CLAUDE.md?
2. What did I correct that Claude got wrong? Should a rule prevent that?
3. What worked well that we should replicate in future sessions?
4. Were any architectural decisions made that CLAUDE.md should record?
5. Is there anything currently in CLAUDE.md that this session proved wrong or outdated?

Output a knowledge feed — 3 to 5 bullets max, high-signal items only.
Skip anything obvious, generic, or already in CLAUDE.md.
Format each item as an actionable rule ready to copy in.
```

## 何时运行

| 触发场景 | 是否运行复盘？ |
|---------|-----------|
| 完成一个功能或重大重构 | 是 |
| 调试会话发现了系统性缺口 | 是 |
| 做出了架构决策 | 是 |
| 新人入职时发现了缺失的上下文 | 是 |
| 每月例行，即使没有大事发生 | 是——捕捉积累性漂移 |
| 琐碎会话（改错别字、编辑文档） | 否 |

即使对于平静的项目，每月运行一次也是最低频率。小幅漂移会悄无声息地积累。

## 好的输出长什么样

Claude 应该返回类似下面的内容（不是一字不差——而是这个颗粒度）：

```
Knowledge Feed — 2025-09-15

1. Pattern established: All database queries go through the repository layer, never called
   directly from route handlers. Rule to add: "Never call Prisma directly from route
   handlers — use the repository functions in src/db/."

2. Correction: I suggested lodash for array manipulation. You redirected to native array
   methods. Rule to add: "Don't import lodash — use native JS array methods. The bundle
   cost isn't worth it for our use case."

3. Architecture decision: We decided to keep auth logic in middleware, not in individual
   route handlers. Rule to add: "Auth checks belong in src/middleware/auth.ts — not in
   route handlers. If a route needs special auth logic, extend the middleware."

4. Stale rule: CLAUDE.md still references the old Express v4 error handler signature.
   We're on v5. Remove: "Express error handlers take 4 arguments (err, req, res, next)."
   Update to: "Express v5 error handlers: async errors propagate automatically — no need
   to call next(err)."
```

## 复盘之后

### 审阅输出内容

不是 Claude 给出的所有内容都适合写进 `CLAUDE.md`。逐条问自己：

- 这是针对我们项目的特定知识，还是 Claude 已知的通用建议？
- 它具有可操作性吗？如果没有这条规则，新的 Claude 会话是否会犯同样的错误？
- 已经有了吗？添加之前先搜索——重复条目会降低遵守率。

### 更新 CLAUDE.md

将新规则添加到最相关的章节。删除 Claude 标记出的过时规则。

### 用可追溯的提交信息提交

```bash
git add CLAUDE.md
git commit -m "context: [改了什么以及为什么]"
```

有意义的提交信息会留下 AI 上下文演进的可追溯历史，方便调试："Claude 在我们做了 Y 更改之后开始把 X 做错了"——从 git log 就能定位。

好的提交信息示例：

```
context: add lodash ban — use native array methods
context: clarify auth middleware pattern after refactor
context: remove Express v4 error handler rule — now on v5
context: document payment webhook idempotency requirement
context: add Prisma-direct query ban after code review
```

## 团队使用

当多人在同一项目中与 Claude 协作时，复盘尤为有价值。不同开发者会收到不同的纠正——`CLAUDE.md` 的不同缺口会从不同角度暴露出来。建议：

- 在任何有大量 Claude 辅助工作的 PR 结束时运行复盘
- 将 `CLAUDE.md` 纳入 PR Review 的常规议程
- 与整个团队每季度进行一次联合复盘，回顾积累的变更

`CLAUDE.md` 是共享资产。它的质量反映了团队维护它的用心程度。
