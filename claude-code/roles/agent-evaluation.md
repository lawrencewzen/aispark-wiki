> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "智能体评估"
description: "衡量自定义智能体效果的指标、模式与工具"
tags: [agents, testing, guide]
---

# 智能体评估

**快速导航**：[为什么评估？](#为什么要评估智能体) · [追踪指标](#追踪指标) · [实施模式](#实施模式) · [示例](#示例带评估的智能体) · [工具](#工具与参考)

---

## 为什么要评估智能体？

当你在 `.claude/agents/` 中创建自定义智能体时，你将专业知识编码为可复用的工作流。但如何知道你的智能体是否真正有效？

**没有评估**，你在摸黑构建：
- ❌ 无法衡量智能体响应随时间是在改善还是下降
- ❌ 无法客观比较不同的智能体配置
- ❌ 难以识别智能体上下文/指令的哪些方面需要改进
- ❌ 没有数据来证明智能体开发投入的合理性

**有了评估**，你可以有信心地迭代：
- ✅ 通过指标量化智能体质量（响应时间、准确性、工具使用）
- ✅ 对不同智能体配置进行可衡量的 A/B 测试
- ✅ 识别成功 vs 失败交互中的模式
- ✅ 建立持续改进的反馈循环

**核心原则**：智能体就是代码。像所有代码一样，它们需要测试、指标和可观测性。

---

## 追踪指标

### 1. 响应质量指标

**衡量什么**：
- **任务完成率**：智能体是否完成了既定目标？
- **正确性**：智能体的输出是否事实准确？
- **相关性**：响应是否保持主题相关并解决实际问题？
- **幻觉率**：智能体发明信息的频率？

**如何追踪**：
```bash
# 工具后钩子：.claude/hooks/log-response-quality.sh
# 每次智能体响应后触发

# 日志结构：
{
  "timestamp": "2026-02-10T14:32:00Z",
  "agent_id": "backend-architect",
  "task_completed": true,
  "correctness_score": 4.5,  # 用户评分 1-5
  "hallucinations": 0,
  "response_tokens": 1250
}
```

**实施提示**：使用用户反馈提示（点赞/点踩）或自动检查（智能体代码生成后的测试套件通过情况）。

---

### 2. 工具使用指标

**衡量什么**：
- **工具调用成功率**：无错误执行的工具调用百分比
- **工具选择准确性**：智能体是否为任务选择了正确的工具？
- **工具调用效率**：实现目标所需的最少调用（避免不必要的读取/搜索）
- **错误恢复**：智能体是否优雅处理工具失败？

**如何追踪**：
```bash
# 工具后钩子：.claude/hooks/log-tool-usage.sh
# 每次工具调用后触发

# 日志结构：
{
  "timestamp": "2026-02-10T14:32:05Z",
  "agent_id": "backend-architect",
  "tool_name": "Read",
  "tool_success": true,
  "tool_parameters": {"file_path": "src/auth.ts"},
  "execution_time_ms": 45
}
```

**实施提示**：使用 Claude Code Hooks 系统（见 `examples/hooks/`）自动记录工具调用。

---

### 3. 性能指标

**衡量什么**：
- **响应时间**：从用户提示词到完整响应的总时间
- **Token（词元）效率**：每个任务使用的输入/输出 Token（词元）数
- **上下文利用率**：使用了多少上下文窗口？
- **每任务成本**：完整交互的 API 成本

**如何追踪**：
```bash
# 会话结束钩子：.claude/hooks/log-performance.sh
# 会话结束时触发

# 日志结构：
{
  "timestamp": "2026-02-10T14:35:00Z",
  "agent_id": "backend-architect",
  "session_duration_s": 180,
  "input_tokens": 3500,
  "output_tokens": 2800,
  "total_cost_usd": 0.15,
  "context_utilization": 0.42
}
```

**实施提示**：解析 Claude Code 会话日志或使用 MCP 可观测性工具。

---

### 4. 用户满意度指标

**衡量什么**：
- **明确反馈**：用户评分、评论、Bug 报告
- **隐式信号**：用户是否接受了智能体的建议？他们是否重新提示？
- **采用率**：相比替代方案，这个智能体的使用频率？
- **留存率**：用户是否会为类似任务回来使用这个智能体？

**如何追踪**：
```bash
# 手动反馈收集
# 智能体完成任务后，提示用户：
"给这个智能体的表现打分（1-5）：_"

# 日志：
{
  "timestamp": "2026-02-10T14:35:10Z",
  "agent_id": "backend-architect",
  "user_rating": 5,
  "user_comment": "完美分析了认证流程",
  "would_use_again": true
}
```

**实施提示**：在智能体模板中添加反馈提示，或使用会话后调查。

---

## 实施模式

### 模式 1：日志 Hook 系统

**用例**：自动追踪所有智能体交互，无需手动干预

**设置**：
```bash
# .claude/hooks/post-tool-use.sh
#!/bin/bash
# 每次工具调用后触发

AGENT_ID=$(echo "$CLAUDE_AGENT_ID" | jq -r)
TOOL_NAME=$(echo "$CLAUDE_TOOL_NAME" | jq -r)
TOOL_SUCCESS=$(echo "$CLAUDE_TOOL_SUCCESS" | jq -r)

# 追加到指标日志
echo "{\"timestamp\":\"$(date -Iseconds)\",\"agent\":\"$AGENT_ID\",\"tool\":\"$TOOL_NAME\",\"success\":$TOOL_SUCCESS}" \
  >> .claude/logs/agent-metrics.jsonl
```

**优点**：零手动开销，完整覆盖，时序数据
**缺点**：需要解析 Claude Code 环境变量（可能跨版本变化）

---

### 模式 2：智能体单元测试

**用例**：回归测试，确保智能体改进不会破坏现有能力

**设置**：
```bash
# tests/agents/backend-architect.test.sh
#!/bin/bash

# 测试 1：智能体正确识别六边形架构层
echo "测试：六边形架构分析"
RESULT=$(claude agent backend-architect "分析 src/auth.ts 的层违规")
if echo "$RESULT" | grep -q "domain layer"; then
  echo "✅ 通过：识别了层次"
else
  echo "❌ 失败：未识别层次"
  exit 1
fi

# 测试 2：智能体推荐正确模式
echo "测试：模式建议"
RESULT=$(claude agent backend-architect "改进 src/api.ts 中的错误处理")
if echo "$RESULT" | grep -q "Result<T, E>"; then
  echo "✅ 通过：推荐了 Result 模式"
else
  echo "❌ 失败：模式不正确"
  exit 1
fi
```

**优点**：自动化，捕获回归，CI/CD 集成
**缺点**：需要维护，可能有误报/漏报

---

### 模式 3：A/B 测试配置

**用例**：比较两个版本的智能体以确定哪个表现更好

**设置**：
```yaml
# .claude/agents/backend-architect-v1.md（对照）
name: backend-architect
version: 1.0
instructions: |
  你是一名专注于...的后端架构师
  [原始指令]

# .claude/agents/backend-architect-v2.md（实验）
name: backend-architect-v2
version: 2.0
instructions: |
  你是一名专注于...的后端架构师
  [修改后的指令，带有新的模式强调]
```

**评估**：
```bash
# 对两个智能体运行相同任务，比较指标
# 任务："分析 src/auth.ts 的安全问题"

# 版本 1 指标：
# - 响应时间：45 秒
# - 发现的问题：3 个
# - 用户评分：4/5

# 版本 2 指标：
# - 响应时间：38 秒
# - 发现的问题：5 个（额外 2 个关键问题）
# - 用户评分：5/5

# 结论：版本 2 更彻底且更快 → 提升到生产
```

**优点**：数据驱动的决策，可量化的改进
**缺点**：需要规范地运行受控实验

---

### 模式 4：反馈循环集成

**用例**：根据实际使用数据持续改进智能体

**设置**：
```bash
# 智能体完成任务后
echo "你如何评价这个响应？（1-5，或'跳过'）："
read RATING

if [ "$RATING" != "skip" ]; then
  echo "有具体反馈吗？："
  read COMMENT

  # 记录反馈
  echo "{\"timestamp\":\"$(date -Iseconds)\",\"agent\":\"$AGENT_ID\",\"rating\":$RATING,\"comment\":\"$COMMENT\"}" \
    >> .claude/logs/agent-feedback.jsonl
fi

# 每周：审查 feedback.jsonl，识别模式
# 每月：根据聚合反馈更新智能体指令
```

**优点**：与实际用户需求对齐，识别边缘案例
**缺点**：需要手动审查和对反馈采取行动

---

## 示例：带评估的智能体

**完整模板可在此获取**：[`examples/agents/analytics-with-eval/`](../../examples/agents/analytics-with-eval/) 包含完整的智能体定义、Hooks、分析脚本和报告模板。

### 设置：带内置指标的分析智能体

```yaml
# .claude/agents/analytics-agent.md
---
name: analytics-agent
description: 带评估 Hooks 的 SQL 查询生成器
version: 1.0
tools:
  - Read
  - Write
  - Bash
hooks:
  post_response: .claude/hooks/log-analytics-metrics.sh
---

# 分析智能体

你是帮助用户查询数据库的专业 SQL 分析师。

## 评估标准

每次查询后：
1. **正确性**：查询是否产生预期结果？
2. **性能**：查询执行时间 < 5 秒？
3. **安全性**：无破坏性操作（DELETE、DROP、TRUNCATE）？
4. **最佳实践**：使用了正确的 JOIN、索引、参数化查询？

## 指令

[... 智能体指令 ...]
```

### 指标 Hook

```bash
# .claude/hooks/log-analytics-metrics.sh
#!/bin/bash
# 分析智能体响应后触发

# 从响应中提取查询（朴素的 grep，可以用 jq 改进）
QUERY=$(echo "$CLAUDE_RESPONSE" | grep -oP 'SELECT.*?;')

if [ -n "$QUERY" ]; then
  # 测试查询（需要数据库连接）
  EXEC_TIME=$( (time psql -U user -d db -c "$QUERY") 2>&1 | grep real | awk '{print $2}')

  # 检查破坏性操作
  if echo "$QUERY" | grep -iE 'DELETE|DROP|TRUNCATE'; then
    SAFETY="FAIL"
  else
    SAFETY="PASS"
  fi

  # 记录指标
  echo "{\"timestamp\":\"$(date -Iseconds)\",\"query\":\"$QUERY\",\"exec_time\":\"$EXEC_TIME\",\"safety\":\"$SAFETY\"}" \
    >> .claude/logs/analytics-metrics.jsonl
fi
```

### 分析

```bash
# 月度审查：分析指标
jq -s 'group_by(.safety) | map({safety: .[0].safety, count: length})' \
  .claude/logs/analytics-metrics.jsonl

# 输出：
# [
#   {"safety": "PASS", "count": 127},
#   {"safety": "FAIL", "count": 3}
# ]

# 行动：审查 3 个失败的查询，更新智能体指令以防止未来违规
```

---

## 工具与参考

### 开源评估框架

#### nao（分析智能体）

**网址**：[github.com/getnao/nao](https://github.com/getnao/nao/)

**提供的内容**：
- 分析智能体的内置评估框架
- 智能体响应的单元测试能力
- 指标收集（响应质量、工具使用、性能）
- 反馈循环集成

**如何适配到 Claude Code**：
- **上下文构建器模式**：将 nao 的结构化上下文方法应用于 `.claude/agents/` 配置
- **评估 Hooks**：将 nao 的评估框架转换为 Claude Code Hooks 系统
- **指标 Schema**：使用 nao 的指标 Schema 作为日志模板

**状态**：生产就绪，积极维护，TypeScript + Python

---

### Claude Code 原生模式

**Hooks 系统**：`.claude/hooks/` 用于自动日志记录（见 `examples/hooks/README.md`）

**智能体目录**：`.claude/agents/` 用于自定义智能体定义（见 `guide/ultimate-guide.md` 第 4 节）

**MCP 可观测性**：使用 MCP 服务器进行高级日志记录和指标聚合

---

## 最佳实践

### 从简单开始

**第 1 周**：添加基本日志 Hook（仅工具调用）
**第 2 周**：添加用户反馈提示（手动评分）
**第 3 周**：构建仪表板以可视化指标
**第 4 周**：对智能体配置运行第一个 A/B 测试

### 专注于可操作指标

不要追踪你不会行动的指标。优先考虑：
1. **任务完成率** → 改进智能体指令
2. **工具调用错误** → 改善上下文或添加示例
3. **用户评分** → 识别令人困惑或无帮助的响应

### 尽可能自动化

手动评估无法扩展。使用：
- Hooks 进行自动日志记录
- CI/CD 集成进行智能体单元测试
- 脚本进行定期指标聚合

### 建立反馈循环

指标如果不采取行动就没用：
- 每周：审查指标，识别模式
- 每月：根据数据更新智能体指令
- 每季度：如有需要进行重大智能体重构

---

## 相关章节

- **[智能体](#4-agents)**：创建自定义智能体
- **[Hooks](#7-hooks)**：使用事件 Hooks 进行自动化
- **[可观测性](../ops/observability.md)**：日志记录和监控策略
- **[AI 生态系统](../ecosystem/ai-ecosystem.md#82-domain-specific-agent-frameworks)**：外部框架（如 nao）

---

**后续步骤**：
1. 为你最常用的智能体添加日志 Hook
2. 收集 1 周的指标
3. 根据数据分析和改进智能体

**模板**：见 `examples/agents/analytics-with-eval/` 获取包含 Hooks、脚本和报告模板的完整实现
