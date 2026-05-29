> 📚 **AI Spark Wiki** · Claude Code 知识库

# 网络防御智能体团队

一条由 4 个智能体组成的流水线，用于检测日志文件中的安全威胁。原生构建于 Claude Code 智能体团队之上。

**本示例旨在对比两种方案**：LangGraph（Python 框架）与 Claude Code 智能体团队（原生方案）。同一个系统，两种架构。差异告诉你各自适用的场景。

---

## 架构

```
log-ingestor (haiku)
      ↓
anomaly-detector (sonnet)
      ↓
risk-classifier (sonnet)
      ↓
threat-reporter (sonnet)
      ↓
cyber-defense-report.md
```

每个智能体职责单一，通过共享 JSON 文件向下一个智能体传递数据。编排技能（`/cyber-defense-team`）负责按顺序生成各智能体并汇总结果。

**用法**：`/cyber-defense-team /var/log/nginx/access.log`

---

## LangGraph vs Claude Code 智能体团队

同一个系统构建了两次。以下是完整对比。

### LangGraph 版本（约 150 行 Python）

```python
from typing import TypedDict, List
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage
import json

# ─── State Definition ─────────────────────────────────────────────────────────
class DefenseState(TypedDict):
    raw_logs: str
    parsed_events: List[dict]
    anomalies: List[dict]
    risk_level: str      # LOW | MEDIUM | HIGH | CRITICAL
    report: str

# ─── Tools (Python functions the LLM can call) ────────────────────────────────
@tool
def detect_patterns(logs: str) -> List[dict]:
    """Extract structured events from raw log text."""
    events = []
    for line in logs.split("\n"):
        if any(k in line for k in ["ERROR", "FAILED", "UNAUTHORIZED"]):
            events.append({"type": "security_event", "raw": line})
        elif "WARNING" in line:
            events.append({"type": "warning", "raw": line})
    return events

@tool
def detect_anomalies(events: List[dict]) -> List[dict]:
    """Detect statistical anomalies in event patterns."""
    anomalies = []
    error_count = sum(1 for e in events if e["type"] == "security_event")
    if error_count > 5:
        anomalies.append({"type": "high_error_rate", "count": error_count, "severity": "HIGH"})
    return anomalies

@tool
def lookup_threat(event_type: str) -> dict:
    """Look up known threat signatures."""
    db = {
        "UNAUTHORIZED": {"description": "Auth bypass attempt"},
        "SQL_INJECTION": {"cve": "CVE-2021-1234", "description": "SQLi pattern"}
    }
    return db.get(event_type, {"description": "Unknown threat"})

# ─── LLM Setup ────────────────────────────────────────────────────────────────
llm = ChatOpenAI(model="gpt-4o-mini")
llm_with_tools = llm.bind_tools([detect_patterns, detect_anomalies, lookup_threat])

# ─── Nodes (one function per agent role) ──────────────────────────────────────
def ingest_node(state: DefenseState) -> DefenseState:
    events = detect_patterns.invoke(state["raw_logs"])
    return {**state, "parsed_events": events}

def detect_node(state: DefenseState) -> DefenseState:
    anomalies = detect_anomalies.invoke(state["parsed_events"])
    return {**state, "anomalies": anomalies}

def classify_node(state: DefenseState) -> DefenseState:
    result = llm_with_tools.invoke([
        SystemMessage("Classify risk as LOW/MEDIUM/HIGH/CRITICAL."),
        HumanMessage(f"Anomalies: {json.dumps(state['anomalies'])}")
    ])
    risk = "HIGH" if state["anomalies"] else "LOW"
    return {**state, "risk_level": risk}

def report_node(state: DefenseState) -> DefenseState:
    result = llm_with_tools.invoke([
        SystemMessage("You are a Senior Security Analyst. Write a Markdown incident report."),
        HumanMessage(f"Risk: {state['risk_level']}\nAnomalies: {json.dumps(state['anomalies'])}")
    ])
    return {**state, "report": result.content}

# ─── Conditional Edge ─────────────────────────────────────────────────────────
def should_report(state: DefenseState) -> str:
    return "report" if state["anomalies"] else END

# ─── Graph Assembly ───────────────────────────────────────────────────────────
builder = StateGraph(DefenseState)
builder.add_node("ingest", ingest_node)
builder.add_node("detect", detect_node)
builder.add_node("classify", classify_node)
builder.add_node("report", report_node)

builder.set_entry_point("ingest")
builder.add_edge("ingest", "detect")
builder.add_edge("detect", "classify")
builder.add_conditional_edges("classify", should_report)
builder.add_edge("report", END)

# ─── Memory ───────────────────────────────────────────────────────────────────
checkpointer = MemorySaver()
app = builder.compile(checkpointer=checkpointer)

# ─── Entry Point ──────────────────────────────────────────────────────────────
def analyze_logs(logs: str) -> str:
    config = {"configurable": {"thread_id": "security-session-1"}}
    result = app.invoke(
        {"raw_logs": logs, "parsed_events": [], "anomalies": [], "risk_level": "", "report": ""},
        config
    )
    return result.get("report", "No threats detected.")
```

### Claude Code 版本（约 60 行 YAML/Markdown）

四个智能体文件，一个技能文件。无需图组装、无 TypedDict、无样板代码。

```
examples/agents/cyber-defense/
├── log-ingestor.md       (~40 行)
├── anomaly-detector.md   (~50 行)
├── risk-classifier.md    (~55 行)
└── threat-reporter.md    (~45 行)

examples/skills/cyber-defense-team/
└── SKILL.md              (~70 行)
```

每个智能体文件由 YAML frontmatter（名称、模型、工具）加上一段描述角色、输入、输出和约束的纯英文系统提示词组成。技能文件通过 `Agent tool` 调用按顺序编排各智能体。

---

## 并排对比

| 维度 | LangGraph | Claude Code 智能体团队 |
|-----------|-----------|------------------------|
| **总代码量** | ~150 行 Python | ~60 行 YAML/Markdown |
| **状态管理** | 显式 `TypedDict` 定义 | 隐式——智能体间共享 JSON 文件 |
| **记忆** | 手动设置 `MemorySaver` | 原生（文件默认持久化） |
| **工具定义** | `@tool` 装饰的 Python 函数 | MCP 服务器或内置工具 |
| **条件逻辑** | `add_conditional_edges()` | 技能中的自然语言（"如果没有异常，跳到第 5 步"） |
| **模型选择** | 所有节点用同一个模型 | 按智能体指定（解析用 haiku，推理用 sonnet） |
| **调试** | `print()` + LangSmith（付费） | Claude Code 原生 UI |
| **新增智能体角色** | 新函数 + `add_node()` + `add_edge()` | 新建一个 `.md` 文件 |
| **上手成本** | 学习 LangGraph API | 读一个 Markdown 文件 |
| **部署** | FastAPI 或 Gradio（再加约 50 行） | `claude` CLI，搞定 |
| **依赖** | `langgraph`、`langchain`、`langchain-openai` | 无（内置于 Claude Code） |

---

## 如何选择

**选 Claude Code 智能体团队，当：**
- 你想快速推进——30 分钟内从原型到可运行系统
- 团队中有非开发人员需要阅读或编辑智能体提示词
- 流水线是内部工具（而非公开 API 端点）
- 需要频繁迭代智能体行为（编辑一个 `.md` 文件即可）

**选 LangGraph，当：**
- 需要将系统嵌入更大的 Python 应用
- 需要将流水线作为 REST API 对外暴露
- 需要可单元测试的确定性状态转换
- 团队以 Python 为主且已有 LangGraph 经验

**真实的权衡**：LangGraph 提供更强的程序化控制和 Python 生态访问能力。Claude Code 智能体团队则减少样板代码、加快迭代速度，且无需维护任何基础设施。对于内部工具和知识型工作流水线，Claude Code 方案在总体拥有成本上通常更占优。

---

## 本示例文件说明

| 文件 | 智能体角色 | 模型 | 职责 |
|------|-----------|-------|----------------|
| `log-ingestor.md` | 阶段 1 | haiku | 解析原始日志 → `cyber-defense-events.json` |
| `anomaly-detector.md` | 阶段 2 | sonnet | 检测模式 → `cyber-defense-anomalies.json` |
| `risk-classifier.md` | 阶段 3 | sonnet | 评估风险 → `cyber-defense-risk.json` |
| `threat-reporter.md` | 阶段 4 | sonnet | 生成报告 → `cyber-defense-report.md` |
| `../skills/cyber-defense-team/SKILL.md` | 编排器 | — | 顺序调度智能体、处理错误、汇总结果 |

---

**灵感来源**：Maryam Miradi 的 SMART COMPASS 框架——同一系统，不同技术栈。
