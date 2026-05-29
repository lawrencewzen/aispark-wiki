> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: cyber-defense-team
description: "编排一个由 4 个智能体组成的网络防御流水线，对日志文件进行威胁分析。适用于调查安全日志、检测访问模式异常、对入侵严重程度分级，或从 nginx/auth/syslog 文件生成事件报告。"
allowed-tools: Read Bash
argument-hint: "[日志文件路径]"
effort: high
metadata:
  version: 1.0.0
---

# 网络防御团队 Skill

编排一个由 4 个智能体组成的流水线，对日志文件进行安全威胁分析，并生成事件报告。

## 流水线架构

```
[You] → Team Lead (this skill)
           │
           ├─[1]─→ log-ingestor    (haiku)  → cyber-defense-events.json
           │
           ├─[2]─→ anomaly-detector (sonnet) → cyber-defense-anomalies.json
           │                                    (reads events.json)
           ├─[3]─→ risk-classifier  (sonnet) → cyber-defense-risk.json
           │                                    (reads anomalies.json)
           └─[4]─→ threat-reporter  (sonnet) → cyber-defense-report.md
                                               (reads all 3 JSON files)
```

第 2、3 阶段顺序执行（每个阶段依赖前一阶段的输出）。第 4 阶段在所有数据就绪后运行。

## 执行步骤

### 第一步 — 验证输入

确认日志文件存在（或已内联提供日志内容）。如果路径不存在，立即告知用户——不要继续执行。

### 第二步 — 启动日志摄取智能体

使用 Agent 工具启动 `log-ingestor` 智能体：

```
Task: Parse the log file at [log_path] and write structured events to cyber-defense-events.json.
Log path: [log_path]
```

等待完成。确认 `cyber-defense-events.json` 已生成。

### 第三步 — 启动异常检测智能体

使用 Agent 工具启动 `anomaly-detector` 智能体：

```
Task: Read cyber-defense-events.json and detect anomalies. Write results to cyber-defense-anomalies.json.
```

等待完成。如果 `anomalies_found: 0`，跳至第五步（报告智能体仍会运行）。

### 第四步 — 启动风险分级智能体

使用 Agent 工具启动 `risk-classifier` 智能体：

```
Task: Read cyber-defense-anomalies.json and classify overall risk. Write result to cyber-defense-risk.json.
```

### 第五步 — 启动威胁报告智能体

使用 Agent 工具启动 `threat-reporter` 智能体：

```
Task: Read cyber-defense-events.json, cyber-defense-anomalies.json, and cyber-defense-risk.json. Generate a complete incident report and save it to cyber-defense-report.md.
```

### 第六步 — 向用户汇总

读取 `cyber-defense-risk.json` 并展示：

```
✅ Analysis complete

Risk Level : HIGH
Score      : 74/100
Threats    : 2 anomalies detected
Report     : cyber-defense-report.md

Primary threat: Brute force attack from 192.168.1.105
Immediate action required: [first recommended_action]
```

## 错误处理

- 第 2 步智能体失败：告知用户，停止流水线，显示原始错误信息。
- 第 3 步及之后智能体失败：展示已有的部分结果，标注哪个阶段失败。
- 日志文件未找到："File [path] not found. Provide a valid path or paste log content."

## 费用估算

| 阶段 | 模型 | 典型 Token 用量 |
|-------|-------|----------------|
| log-ingestor | haiku | ~2K |
| anomaly-detector | sonnet | ~3K |
| risk-classifier | sonnet | ~2K |
| threat-reporter | sonnet | ~3K |
| **合计** | | **~10K** |

对于大型日志文件（超过 10K 行），log-ingestor 最多可能消耗 20K Token。

## 使用示例

```
/cyber-defense-team /var/log/nginx/access.log
/cyber-defense-team /tmp/auth.log
```
