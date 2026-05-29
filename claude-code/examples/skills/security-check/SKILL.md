> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: security-check
description: 快速对照已知威胁数据库进行配置安全检查
argument-hint: "[path]"
effort: low
disable-model-invocation: true
---

# 安全检查

快速对照已知威胁数据库进行配置安全检查。验证你的 Claude Code 配置中是否存在已知恶意技能、有漏洞的 MCP、危险模式及暴露的密钥。

**时间**：约30秒 | **范围**：仅限 Claude Code 配置

## 说明

你是一名安全分析师。根据打包在 `examples/skills/update-threat-db/threat-db.yaml` 中的威胁情报数据库，检查用户的 Claude Code 配置。生成简洁、可操作的报告。

### 第1阶段：加载威胁数据库

从本仓库读取 `examples/skills/update-threat-db/threat-db.yaml`，加载：
- 已知恶意作者和技能
- MCP 服务器的 CVE 数据库
- 钩子、智能体和配置的可疑模式

### 第2阶段：MCP 服务器审计

读取用户的 MCP 配置：

```bash
# Global MCP config
cat ~/.claude.json 2>/dev/null | jq '.mcpServers // empty'

# Project MCP config
cat .mcp.json 2>/dev/null
```

**对照 threat-db.yaml 检查：**
- [ ] 是否有 MCP 服务器匹配 CVE 条目？→ CRITICAL
- [ ] 版本锁定：所有 MCP 服务器是否锁定为精确版本（而非 `@latest`）？→ 未锁定则为 HIGH
- [ ] MCP 参数中是否有 `--dangerous-*` 标志？→ CRITICAL
- [ ] 是否有不在安全列表中的 MCP 服务器（见 `guide/security-hardening.md` §1.1）？→ MEDIUM（标记供人工审查）

### 第3阶段：技能与智能体审计

```bash
# List installed skills
ls -la .claude/skills/ 2>/dev/null
ls -la ~/.claude/skills/ 2>/dev/null

# List agents
ls -la .claude/agents/ 2>/dev/null
ls -la ~/.claude/agents/ 2>/dev/null

# Check agent tools field
grep -r "^tools:" .claude/agents/ 2>/dev/null
grep -r "^tools:" ~/.claude/agents/ 2>/dev/null
```

**对照 threat-db.yaml 检查：**
- [ ] 是否有技能/智能体名称匹配 `malicious_skills` 条目？→ CRITICAL
- [ ] 是否有技能/智能体作者匹配 `malicious_authors` 条目？→ CRITICAL
- [ ] 是否有仅配置 `tools: Bash` 的智能体？→ HIGH
- [ ] 是否有工具访问权限过宽且描述模糊的智能体？→ MEDIUM

### 第4阶段：钩子安全

```bash
# List all hooks
find .claude/hooks/ -type f 2>/dev/null
find ~/.claude/hooks/ -type f 2>/dev/null

# Scan hooks for suspicious patterns
grep -rn "curl\|wget\|nc \|ncat\|netcat\|base64\|eval\|exec\|/dev/tcp\|/dev/udp" .claude/hooks/ 2>/dev/null
grep -rn "curl\|wget\|nc \|ncat\|netcat\|base64\|eval\|exec\|/dev/tcp\|/dev/udp" ~/.claude/hooks/ 2>/dev/null

# Check for credential access in hooks
grep -rn "ssh\|id_rsa\|id_ed25519\|\.env\|credentials\|secret\|password\|token\|api.key" .claude/hooks/ 2>/dev/null
grep -rn "ssh\|id_rsa\|id_ed25519\|\.env\|credentials\|secret\|password\|token\|api.key" ~/.claude/hooks/ 2>/dev/null
```

**对照 threat-db.yaml `suspicious_patterns.hooks` 检查：**
- [ ] 网络调用（`curl`、`wget`）→ HIGH
- [ ] 反向 shell 指示器（`nc`、`/dev/tcp`）→ CRITICAL
- [ ] 凭证访问（`ssh`、`.env`、`password`）→ CRITICAL
- [ ] Base64 编码 → MEDIUM（审查上下文）

### 第5阶段：记忆投毒检查

```bash
# Check for suspicious instructions in memory/config files
grep -in "ignore\|forget\|override\|disregard\|you are now\|new role\|system prompt" \
  CLAUDE.md .claude/CLAUDE.md SOUL.md .claude/SOUL.md MEMORY.md .claude/MEMORY.md \
  ~/.claude/CLAUDE.md ~/.claude/MEMORY.md 2>/dev/null
```

- [ ] CLAUDE.md / SOUL.md / MEMORY.md 中是否存在提示词注入模式？→ HIGH
- [ ] 是否有禁用安全机制、跳过审查或授予宽泛权限的指令？→ CRITICAL

### 第6阶段：权限与设置

```bash
# Check settings
cat .claude/settings.json 2>/dev/null
cat ~/.claude/settings.json 2>/dev/null
```

- [ ] `permissions.deny` 是否存在且覆盖了 `.env*`、`*.pem`、`*.key`、密钥等？→ 缺失则为 MEDIUM
- [ ] Bash 或 Write 的 `permissions.allow` 中是否有通配符？→ 存在则为 HIGH
- [ ] 是否有 `dangerouslySkipPermissions` 或类似标志？→ 存在则为 CRITICAL

### 第7阶段：配置中暴露的密钥

```bash
# Check for secrets in .claude/ directory
grep -rn "sk-[a-zA-Z0-9]\{20,\}\|sk-ant-[a-zA-Z0-9]\{20,\}\|ghp_[a-zA-Z0-9]\{36\}\|AKIA[A-Z0-9]\{16\}" \
  .claude/ ~/.claude/ 2>/dev/null

# Check for private keys
grep -rn "BEGIN.*PRIVATE KEY" .claude/ ~/.claude/ 2>/dev/null
```

- [ ] 配置文件中是否有 API 密钥或令牌？→ CRITICAL
- [ ] 配置中是否有私钥？→ CRITICAL

## 输出格式

```
## 🛡️ Security Check Report

**Date**: [timestamp]
**Scope**: Claude Code configuration

### Results Summary

| Severity | Count | Status |
|----------|-------|--------|
| 🔴 CRITICAL | X | [PASS/FAIL] |
| 🟠 HIGH | X | [PASS/FAIL] |
| 🟡 MEDIUM | X | [PASS/FAIL] |
| 🟢 LOW | X | [PASS/FAIL] |

### 🔴 Critical Issues
[List each critical finding with location and fix]

### 🟠 High Issues
[List each high finding with location and fix]

### 🟡 Medium Issues
[List each medium finding with location and fix]

### ✅ Passed Checks
[List what passed — important for confidence]

### 🔧 Recommended Actions (Priority Order)
1. [Most urgent fix with exact command]
2. [Second priority]
3. [...]

### 📚 References
- Full security guide: guide/security-hardening.md
- Threat database: examples/skills/update-threat-db/threat-db.yaml
- MCP scan: `npx mcp-scan` (Snyk)
```

若所有检查均通过，则输出：

```
## 🛡️ Security Check Report — ALL CLEAR ✅

**Date**: [timestamp]
No known threats detected in your Claude Code configuration.

**Recommendations for continued security:**
- Re-run `/security-check` after installing new skills or MCP servers
- Run `/security-audit` for a comprehensive project + config audit
- Keep Claude Code updated (current security fixes in v2.1.34+)
```

$ARGUMENTS
