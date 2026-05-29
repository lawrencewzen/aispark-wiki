> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: investigate
description: 系统性根因调试 — 在编写任何修复代码之前先找到根本原因
argument-hint: <issue_description>
effort: medium
disable-model-invocation: true
---

# Investigate — 根因调试

系统性调试，在做任何代码修改之前，必须强制完成根因调查。

**铁律：未查明根因，禁止修复任何问题。**

修复症状会导致"打地鼠"式调试。每一次未针对根因的修复，都会让下一个 bug 更难找到。

## 说明

### 第一阶段：收集症状

在形成任何假设之前，先收集所有可用上下文。

1. 完整阅读错误信息、堆栈跟踪和复现步骤
2. 如果用户提供的上下文不足，提一个针对性问题：
   - "你看到的确切错误信息是什么？"
   - "这个问题能稳定复现吗？"
   - "这个问题是什么时候开始出现的？"
3. 确定受影响的组件及其边界

**输出**：精确的症状描述 — 什么出错了、何时出错、报什么错。

---

### 第二阶段：阅读代码

从症状出发，沿代码路径追溯到可能的根因。不要猜测。

```bash
# Find all references to the failing component
grep -rn "ComponentName\|function_name\|error_string" src/ --include="*.{ts,js,py,rb,go}" | head -30

# Check recent changes to affected files
git log --oneline -15 -- <affected-file>

# Read the actual diff for each recent commit
git show <commit-hash> -- <affected-file>
```

用 Grep 找到所有引用，用 Read 理解逻辑。不要跳过阅读代码这一步。

---

### 第三阶段：检查近期变更

```bash
# What changed recently across the whole repo
git log --oneline -20

# Changes to files related to the symptom
git log --oneline -20 -- <affected-files>

# Full diff of the last N commits
git diff HEAD~3..HEAD -- <affected-directory>
```

**关键问题**：这个功能之前是正常的吗？如果是，根因就在近期的 diff 里。

- 回归 = 根因在变更内容中，而非原始代码
- 一直损坏 = 架构问题或错误假设

---

### 第四阶段：复现

在修复任何问题之前，先确认你能确定性地触发这个 bug。

```bash
# Run the test suite targeting the affected area
npm test -- --testPathPattern="affected-module" 2>/dev/null || \
pnpm test -- --testPathPattern="affected-module" 2>/dev/null || \
pytest tests/test_affected.py -v 2>/dev/null

# Check logs if available
tail -50 logs/error.log 2>/dev/null || \
journalctl -u app-service --lines=50 2>/dev/null
```

如果无法复现：收集更多证据。不要去修复你无法验证确实损坏的问题。

---

### 第五阶段：模式分析

将症状与已知 bug 模式进行对照：

| 模式 | 特征信号 | 排查位置 |
|---------|-----------|---------------|
| 竞态条件 | 间歇性、依赖时序的失败 | 并发访问共享状态、async/await 顺序 |
| null 传播 | TypeError、undefined is not a function | 可选值缺少守卫、未检查的 API 响应 |
| 状态损坏 | 数据不一致、部分更新 | 事务、回调、共享对象的直接修改 |
| 集成失败 | 超时、响应结构不符合预期 | 外部 API 调用、服务边界、schema 变更 |
| 配置漂移 | 本地正常、预发/生产失败 | 环境变量、功能开关、数据库状态、缺失密钥 |
| 缓存过期 | 显示旧数据、重启/清除后恢复 | Redis、CDN、浏览器缓存、memoization |
| 模块导入错误 | "Cannot find module"、"is not a function" | 包版本、循环导入、构建产物 |

还需检查：
- `TODOS.md` 或 issue 追踪系统，查找同一区域的已知问题
- `git log`，查看相同文件的历史修复记录 — 同一位置反复出现的 bug 是架构异味

**外部搜索：** 如果模式不匹配，搜索：
`{框架} {清理后的错误类型}` — 去掉主机名、文件路径、内部数据。搜索错误类别，而非原始错误信息。

**形成假设**："根因假设：[关于出错原因的具体、可验证的论断]"

---

### 第六阶段：假设验证

在编写任何修复代码之前，先验证你的假设。

1. **确认假设**：在疑似根因位置添加临时日志语句、断言或调试输出。运行复现步骤。证据吻合吗？

```javascript
// Example: temporary diagnostic
console.log('[DEBUG investigate]', { value, expected, type: typeof value });
```

```python
# Example: temporary diagnostic
import sys; print(f'[DEBUG investigate] value={value!r} type={type(value)}', file=sys.stderr)
```

2. **如果假设错误**：收集更多证据，返回第二阶段。不要猜测。

3. **三次失败规则**：如果 3 个假设均未得到验证，立即停止。这可能是架构问题。

   向用户展示：
   ```
   已测试 3 个假设，均未得到确认。这可能需要更深入的调查。

   选项：
   A) 我有新的假设：[描述] — 继续调查
   B) 添加埋点并等待 — 下次捕捉 bug 现场
   C) 上升处理 — 需要对系统有更深了解的人介入
   ```

**红色警告 — 立即放慢脚步：**
- "暂时先这样修" — 不存在"暂时"
- 在追踪数据流之前就提出修复方案 — 那是在猜测
- 每次修复都暴露出另一个新问题 — 层次选错了，不是代码写错了

---

### 第七阶段：实施

根因确认后：

1. **修复根因，而非症状。** 能消除实际问题的最小改动。

2. **最小 diff**：触及文件数最少，修改行数最少。抵制顺手重构周边代码的冲动。

3. **编写回归测试**，要求：
   - 没有修复时**失败**（证明测试有意义）
   - 有了修复后**通过**（证明修复有效）

4. **运行完整测试套件**并粘贴输出结果。不允许引入新的回归。

5. **影响范围检查**：如果修复涉及超过 5 个文件，停下来确认：

   ```
   此修复涉及 N 个文件。对于一个 bug 修复来说，影响范围相当大。

   A) 继续 — 根因确实跨越这些文件
   B) 拆分 — 现在修复关键路径，延后更广泛的清理
   C) 重新思考 — 可能存在更精准的方案
   ```

---

## 输出格式

```
调试报告
════════════════════════════════════════════════════
症状：          [用户观察到的现象]
根因：          [实际出错的原因 — 具体，不模糊]
修复：          [修改了什么，附文件:行号引用]
证据：          [测试输出或复现结果，证明修复有效]
回归测试：      [新测试的文件:行号]
相关：          [已知问题、同区域历史 bug、架构说明]
状态：          DONE | DONE_WITH_CONCERNS | BLOCKED
════════════════════════════════════════════════════
```

**状态定义：**
- **DONE** — 根因已找到，修复已应用，回归测试已编写，所有测试通过
- **DONE_WITH_CONCERNS** — 已修复但无法完全验证（间歇性问题，需要预发环境）
- **BLOCKED** — 完整调查后根因仍不明确

**阻塞时的上升格式：**
```
状态：BLOCKED
原因：[1-2 句话说明尝试了什么以及为何失败]
已尝试：[已测试的假设列表]
建议：[用户下一步应该做什么 — 添加日志、上升处理、架构评审]
```

## 重要规则

- **永远不要应用你无法验证的修复。** 如果无法复现并确认，就不要上线。
- **永远不要说"这应该能修好"。** 验证并证明它。运行测试。
- **一次调查中不要修复超过 3 个不相关的问题。** 如果发现其他 bug，记录下来但保持专注。
- **提交修复前删除所有调试日志语句。**
- **3 个以上假设失败 → 质疑架构**，而不是质疑你的假设能力。

## 用法

```
/investigate TypeError: Cannot read properties of undefined (reading 'map')
/investigate the payment flow is silently dropping some transactions
/investigate
```

## 相关命令

- `/review-pr` — 合并前审查修复
- `/qa` — 修复后对受影响功能进行浏览器 QA
- `/ship` — 调查完成后的预部署清单

$ARGUMENTS
