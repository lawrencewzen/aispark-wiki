> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: token-audit
description: "审计 Claude Code 配置，测量固定上下文的 token 开销并生成优先级行动计划。适用于触及速率限制、上下文过早压缩，或添加了大量配置文件之后。"
effort: medium
allowed-tools: Read Grep Glob Bash
---

# /token-audit — 上下文 Token 审计

**目的**：测量 Claude Code 配置在任何用户任务开始前消耗了多少 token。识别最大的开销来源，并生成包含节省估算的具体行动计划。

**适用场景**：
- 一天结束前就触及了速率限制
- 会话感觉迟缓或上下文过早压缩
- 添加了大量规则文件，想知道实际消耗
- 完成重大配置变更之后

---

## 你将测量的内容

| 组件 | 加载时机 | 典型范围 |
|-----------|-------------|---------------|
| `~/.claude/CLAUDE.md` + @imports | 始终 | 5-15K tokens |
| 项目 `CLAUDE.md` | 始终 | 2-8K tokens |
| `.claude/rules/*.md` | 始终（所有文件） | 5-40K tokens |
| `MEMORY.md` | 始终 | 1-3K tokens |
| Claude Code 系统提示词 | 始终 | ~7,500 tokens |
| 钩子 stdout | 每次工具调用 | 不定 |
| 命令、智能体、技能 | 仅在调用时 | 默认为 0 |

核心洞察：`.claude/rules/` 会在会话启动时加载所有 `.md` 文件，无论其是否相关。命令和智能体采用懒加载——调用前零消耗。规则文件是最常见的意外开销来源。

---

## 第一步 — 执行测量

在项目根目录执行以下命令：

```bash
# 各组件大小
echo "=== PROJECT CLAUDE.md ===" && wc -c CLAUDE.md 2>/dev/null || echo "none"

echo ""
echo "=== RULES FILES (sorted by size) ===" && find .claude/rules -name "*.md" 2>/dev/null \
  | xargs wc -c 2>/dev/null | sort -rn | head -20

echo ""
echo "=== GLOBAL ~/.claude ===" && ls -la ~/.claude/*.md 2>/dev/null \
  | awk '{print $5, $9}' | sort -rn
```

然后计算完整预算：

```bash
GLOBAL=$(cat ~/.claude/CLAUDE.md ~/.claude/*.md 2>/dev/null | wc -c)
PROJECT=$(wc -c < CLAUDE.md 2>/dev/null || echo 0)
RULES=$(find .claude/rules -name "*.md" 2>/dev/null | xargs cat 2>/dev/null | wc -c || echo 0)
MEMORY=$(find ~/.claude/projects -name "MEMORY.md" 2>/dev/null \
  | xargs grep -l "$(basename $(pwd))" 2>/dev/null | head -1 \
  | xargs wc -c 2>/dev/null | awk '{print $1}' || echo 0)
TOTAL=$(( GLOBAL + PROJECT + RULES + MEMORY + 30000 ))

echo "全局 ~/.claude     : ~$(( GLOBAL / 4 )) tokens ($(( GLOBAL / 1000 ))K chars)"
echo "项目 CLAUDE.md     : ~$(( PROJECT / 4 )) tokens"
echo "规则（自动加载）   : ~$(( RULES / 4 )) tokens"
echo "MEMORY.md          : ~$(( MEMORY / 4 )) tokens"
echo "系统提示词         : ~7,500 tokens"
echo "---"
echo "固定上下文总计     : ~$(( TOTAL / 4 )) tokens"
echo "占 200K 窗口比例   : $(( TOTAL / 4 * 100 / 200000 ))%"
```

---

## 第二步 — 对规则文件分类

对 `.claude/rules/` 中的每个文件进行分类：

| 类别 | 定义 | 操作 |
|-------|------------|--------|
| **ALWAYS（始终）** | 适用于大多数任务（规范、输出格式、安全） | 保持自动加载 |
| **SOMETIMES（有时）** | 在 20-40% 的会话中相关 | 文件小则保留（<3K chars）；文件大则改为懒加载 |
| **RARELY（很少）** | 在 <10% 的会话中相关（Figma、Windows、设计系统） | 从自动加载中移除 |
| **NEVER（从不）** | 已过时或已被其他地方覆盖 | 删除或归档 |

执行以下分类提示词：

```
Read every file in .claude/rules/. For each file, output a table row:

| File | Size (chars) | Class (ALWAYS/SOMETIMES/RARELY/NEVER) | Reasoning (one sentence) |

Sort by size descending within each class.
At the end, calculate: total chars that would leave the fixed context if all
RARELY and NEVER files were excluded. Convert to tokens (÷ 4).
```

---

## 第三步 — 审计钩子开销

`PreToolUse` 和 `PostToolUse` 上的钩子在每次工具调用时触发，每次调用都会将其 stdout 注入上下文。一个钩子在每次会话 150 次工具调用中输出 500 个字符，就会产生 75K chars ≈ 19K 额外 token。

检查已有的钩子：

```bash
# 按事件类型列出钩子
python3 - << 'EOF'
import json, os
for path in [os.path.expanduser("~/.claude/settings.json"), ".claude/settings.json"]:
    if not os.path.exists(path): continue
    print(f"\n--- {path} ---")
    data = json.load(open(path))
    for event, hooks in data.get("hooks", {}).items():
        for h in hooks:
            cmd = h.get("command", "?")
            matcher = h.get("matcher", "*")
            print(f"  [{event}] matcher={matcher} → {cmd[:80]}")
EOF
```

对于每个 `PreToolUse` 或 `PostToolUse` 钩子，手动运行以估算其 stdout 大小，再乘以每次会话的平均工具调用次数（可在会话结束后通过 `/cost` 查看）。

**危险信号**：
- 无条件 `cat` 文件的钩子
- 每次调用都执行 `git status` 或 `git log`
- 调试用的多行 echo 输出从未被移除
- 将 JSON 块注入上下文

---

## 第四步 — 制定行动计划

生成一个优先级表。经验法则：只纳入无需外部基础设施即可完成的行动（不涉及 RAG、向量数据库、自定义 MCP 服务器）。

| 操作 | 预计节省 token | 工作量 | 风险 |
|--------|------------------------|--------|------|
| 将 RARELY 文件移出自动加载 | 因情况而异 | 30 分钟 | 低 |
| 将大规则文件拆分为核心 + 详情 | 因情况而异 | 1-2 小时 | 低 |
| 精简钩子 stdout 至必要字段 | 因情况而异 | 1 小时 | 低 |
| 压缩冗长规则（参见 §8 context-engineering.md） | 规则减少 20-30% | 1-2 小时 | 低 |
| 归档过时的 MEMORY.md 条目 | 500-1K tokens | 30 分钟 | 低 |

---

## 第五步 — RAG 的必要性评估

懒加载向量数据库（RAG）有时被作为解决方案推荐。在投入之前，请客观评估：

1. 完成第 1-4 步后，固定上下文 token 还剩多少？（先测量。）
2. RAG 是否合理？pgvector + 自定义 MCP 的搭建需要 1-2 周。
3. 盈亏平衡点：如果你有 10 个平均 3K chars 的规则文件，分类整理（30 分钟）能节省的量与 RAG 相当。当规则文件超过 50 个、只有基于意图的路由才能规模化时，RAG 才值得其代价。

---

## 输出格式

审计完成后，生成以下报告：

```markdown
## Token 审计 — [项目] — [日期]

### 预算摘要

| 组件 | Tokens | 占总量百分比 |
|-----------|--------|------------|
| 全局 ~/.claude | X | Y% |
| 项目 CLAUDE.md | X | Y% |
| 规则（自动加载） | X | Y% |
| MEMORY.md | X | Y% |
| 系统提示词 | 7,500 | Y% |
| **总计** | **X** | **100%** |

任何任务开始前已占用上下文窗口：X%（共 200K）

### 规则分类

| 文件 | 字符数 | 类别 | 操作 |
|------|-------|-------|--------|
| ... | ... | ALWAYS/SOMETIMES/RARELY | 保留/懒加载/移除 |

### 钩子开销

| 钩子 | 事件 | 预计 stdout | 每次会话调用次数 | 每次会话 token 总量 |
|------|-------|-------------|---------------|----------------------|
| ... | PreToolUse | X chars | ~Y | ~Z tokens |

### 行动计划

| 操作 | 节省量 | 工作量 | 风险 |
|--------|---------|--------|------|
| ... | -X tokens | 30 分钟 | 低 |

**无需基础设施可实现的总节省量**：-X tokens → 从 Y 降至 Z（减少 N%）

### RAG 裁决

[一段话：行动计划执行后的剩余开销、RAG 是否合理、预估搭建成本与节省量的对比。]
```

---

## 解读结果

| 固定上下文 | 评估 |
|---------------|------------|
| < 20K tokens | 健康——无需紧急处理 |
| 20-40K tokens | 中等——执行分类整理，抓取容易实现的改进 |
| 40-60K tokens | 较高——规则审计值得花半天时间 |
| > 60K tokens | 严重——任何任务开始前你已消耗 30%+ 的上下文窗口 |

对于配置较重的项目，首次审计后通常可实现 48% 的缩减，且无需任何基础设施变更——仅靠将 RARELY 文件移出自动加载即可。
