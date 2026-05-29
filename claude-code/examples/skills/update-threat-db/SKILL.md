> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: update-threat-db
description: 研究并更新 AI 智能体安全威胁情报数据库
argument-hint: "[--source <url>]"
effort: high
disable-model-invocation: true
---

# 更新威胁数据库

研究并更新 AI 智能体安全威胁情报数据库，收录最新威胁、CVE、恶意 skill 及攻击活动。

**耗时**：3-8 分钟 | **范围**：`examples/skills/update-threat-db/threat-db.yaml`

> 需要 Perplexity MCP（或手动网络搜索）。建议每月运行一次，或在重大安全公告发布后运行。

## 操作说明

你是一名专注于 AI 编码智能体安全的威胁情报分析师。请研究最新威胁并更新威胁数据库。

---

### 第一阶段：现状评估

读取当前威胁数据库：

```
Read examples/skills/update-threat-db/threat-db.yaml
```

记录：
- 当前 `version` 及 `updated` 日期
- 恶意作者、skill、CVE、攻击活动的数量
- 最近的条目，以避免重复录入

---

### 第二阶段：研究新威胁

执行 **4 次定向 Perplexity 搜索**（尽量并行）：

**搜索 1：新恶意 skill 与攻击活动**
```
查询词："malicious AI agent skills ClawHub OpenClaw skills.sh 2026 new campaigns malware supply chain"
重点：threat-db.yaml 中尚未收录的新恶意 skill 名称、作者及攻击活动
```

**搜索 2：新 MCP 服务器 CVE**
```
查询词："MCP server CVE vulnerability 2025 2026 model context protocol security advisory"
重点：MCP 服务器的新 CVE、SDK 漏洞、传输层缺陷
```

**搜索 3：新攻击技术**
```
查询词："AI coding agent attack prompt injection Claude Code Cursor supply chain security research 2026"
重点：新攻击向量、攻击技术、研究论文
```

**搜索 4：新防御工具与黑名单**
```
查询词："MCP security scanner tool mcp-scan alternative AI agent skills security scanning 2026"
重点：新扫描工具、黑名单、防御框架
```

若 Perplexity MCP 不可用，请对每条查询改用 WebSearch。

---

### 第三阶段：分析与去重

对第二阶段的每项发现：

1. **检查是否已在 threat-db.yaml 中** — 跳过重复项
2. **核实来源可信度** — 优先采信：CVE 数据库、安全厂商博客、同行评审研究
3. **分类归档** — 属于哪个分类？
   - `malicious_authors` — 新确认的恶意发布者
   - `malicious_skills` — 新确认的恶意 skill/包名
   - `malicious_skill_patterns` — 用于通配符匹配的新前缀模式
   - `cve_database` — 含组件、严重性、修复版本的新 CVE
   - `minimum_safe_versions` — 若有新补丁则更新最低安全版本
   - `iocs` — 新 C2 IP、数据外泄 URL、恶意软件哈希
   - `campaigns` — 新的协同攻击活动
   - `attack_techniques` — 新记录的攻击向量
   - `scanning_tools` — 新工具或重大更新
   - `defensive_resources` — 新框架、黑名单

4. **评估风险等级**：
   - `critical` — 已确认为恶意，正在被主动利用
   - `high` — 已确认存在漏洞，且有可用的利用方式
   - `medium` — 理论风险，尚无已知利用
   - `low` — 仅供参考

---

### 第四阶段：更新 threat-db.yaml

按以下规则应用变更：

1. **升级版本号** — 新增条目时递增次版本号（如 2.0.0 → 2.1.0），Schema 变更时递增主版本号
2. **更新 `updated` 日期** — 设为今天
3. **添加新来源** — 将新研究来源加入 `sources` 列表
4. **保持 YAML 合法性** — 包含反斜杠的模式使用单引号
5. **保留现有条目** — 除非确认为误报，否则不删除任何条目
6. **遵循现有格式** — 与已有条目的结构完全一致

**重要**：编辑完成后，验证 YAML 合法性：
```bash
python3 -c "import yaml; yaml.safe_load(open('examples/skills/update-threat-db/threat-db.yaml')); print('YAML valid')"
```

---

### 第五阶段：更新依赖文件（如需）

检查新 CVE 是否也应添加到安全加固指南：

```bash
# 对比 threat-db 与 security-hardening 中的 CVE 数量
grep -c "id:" examples/skills/update-threat-db/threat-db.yaml
grep -c "CVE-" guide/security-hardening.md
```

若发现重大新 CVE（严重性为 critical/high）：
- 考虑添加到 `guide/security-hardening.md` 的 CVE 表格
- 若已发布新补丁，更新 `minimum_safe_versions`

---

### 第六阶段：汇总报告

## 输出格式

```
## 威胁数据库更新报告

**日期**：[时间戳]
**旧版本**：[旧版本号]
**新版本**：[新版本号]

### 变更摘要

| 分类 | 新增 | 更新 | 合计 |
|----------|-------|---------|-------|
| 恶意作者 | +X | ~X | XX |
| 恶意 skill | +X | ~X | XX |
| CVE | +X | ~X | XX |
| 攻击活动 | +X | ~X | XX |
| IOC | +X | ~X | XX |
| 攻击技术 | +X | ~X | XX |
| 扫描工具 | +X | ~X | XX |

### 新增条目

[列出每条新条目，注明来源和风险等级]

### 重要发现

[重点说明特别重要或紧迫的内容]

### 无需变更

[若未发现新内容，说明已搜索的范围并确认数据库为最新状态]

### 后续步骤

- [ ] 运行 `/security-check` 以测试更新后的数据库
- [ ] 若有新的严重 CVE，更新 `guide/security-hardening.md`
- [ ] 提交：`docs(security): update threat-db vX.Y.Z — [摘要]`
```

$ARGUMENTS
