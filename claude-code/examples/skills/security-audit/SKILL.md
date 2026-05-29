> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: security-audit
description: 全面安全审计，含安全态势评分
argument-hint: "[path] [--owasp] [--verbose]"
effort: high
disable-model-invocation: true
---

# 安全审计

对项目及 Claude Code 配置进行全面安全审计。分析密钥暴露、注入面、依赖项、钩子安全性，并生成带评分的安全态势评估报告。

**耗时**：2-5 分钟 | **范围**：完整项目 + Claude Code 配置

> 如需仅检查配置，请使用 `/security-check`。

## 操作说明

你是一名资深应用安全工程师。执行 6 阶段安全审计，并生成带有优先级修复计划的评分报告。

---

### 前置步骤：建立审计上下文

**在运行任何检查之前**，使用 `AskUserQuestion` 询问：

1. **环境**：此代码是运行在生产环境、预发布环境还是本地开发环境？
2. **范围**：完整审计还是优先检查特定领域？

这对于准确识别问题至关重要：
- **本地开发**：`DEBUG=True`、CORS `*`、无 TLS 的 HTTP、`.env` 文件 — 均属正常。**不要**标记为漏洞，改为在"上线前注意"信息性章节中提及。
- **预发布**：配置应与生产环境保持一致，偏差标记为 MEDIUM。
- **生产**：任何配置错误均为真实发现，按完整严重级别处理。

如果用户未作答或不确定，默认按**生产环境**处理（保守策略）。

---

### 阶段 1：配置安全（通过 /security-check）

执行 `/security-check`（即 `examples/skills/security-check/SKILL.md` 命令）中的所有检查，涵盖：
- 对照 CVE 数据库审计 MCP 服务器
- 对照已知恶意条目审计技能和智能体
- 钩子数据泄露模式检测
- 记忆污染检测
- 权限与设置审查
- Claude Code 配置中的密钥暴露检查

记录检查结果 — 其将计入最终评分。

---

### 阶段 2：项目密钥扫描

扫描整个项目中暴露的密钥和凭据：

```bash
# API 密钥和令牌
grep -rn --include="*.{js,ts,py,go,java,rb,php,yaml,yml,json,toml,env,cfg,ini,conf}" \
  -E '(?i)(api[_-]?key|apikey|secret|password|passwd|token|bearer|auth)\s*[=:]\s*["'\''"][^"'\'']{8,}["'\''"]\s' \
  --exclude-dir={node_modules,vendor,.git,dist,build,target,__pycache__,.venv} . 2>/dev/null | head -30

# 已知提供商密钥模式
grep -rn -E 'sk-[a-zA-Z0-9]{20,}|sk-ant-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36}|AKIA[A-Z0-9]{16}|xox[bps]-[a-zA-Z0-9\-]{20,}' \
  --exclude-dir={node_modules,vendor,.git,dist,build,target} . 2>/dev/null | head -20

# 私钥
grep -rn 'BEGIN.*PRIVATE KEY' --exclude-dir={node_modules,vendor,.git} . 2>/dev/null

# 可能已提交的 .env 文件
find . -name ".env*" -not -path "*/node_modules/*" -not -path "*/.git/*" -type f 2>/dev/null

# 检查 .gitignore 覆盖情况
[ -f ".gitignore" ] && {
  grep -q "\.env" .gitignore && echo "✅ .env in .gitignore" || echo "⚠️ .env NOT in .gitignore"
  grep -q "\.pem" .gitignore && echo "✅ .pem in .gitignore" || echo "⚠️ .pem NOT in .gitignore"
  grep -q "\.key" .gitignore && echo "✅ .key in .gitignore" || echo "⚠️ .key NOT in .gitignore"
}
```

**防误报规则 — 报告任何密钥问题前必须执行：**

在提出密钥问题之前，先运行以下验证命令：

```bash
# 1. 验证 .env 是否在 .gitignore 中（若是，本地 .env 不构成问题）
grep -n '\.env' .gitignore 2>/dev/null || echo ".env NOT in .gitignore"

# 2. 验证密钥是否真正被提交（空输出 = 无问题）
git log --all -p -- '*.env' '*.key' '*.pem' '*.secret' 2>/dev/null | grep -E '^\+.*(password|secret|api_key|token)' | head -20

# 3. 在 git 历史中检查提供商特定模式
git log --all -p 2>/dev/null | grep -E '^\+(sk-[a-zA-Z0-9]{20,}|AKIA[A-Z0-9]{16}|ghp_[a-zA-Z0-9]{36})' | head -10
```

只有在**以上命令提供具体证据**时才报告密钥问题。本地存在的 `.env` 文件若已在 `.gitignore` 中，则不构成问题。不要仅凭模式匹配就报告"密钥可能暴露"。

**评分：**
- 未发现密钥 → +20 分
- 发现 1-3 个 → +10 分
- 发现 4 个以上 → +0 分
- 私钥已被提交 → -10 分

---

### 阶段 3：提示词注入面

分析 Markdown 和配置文件中的注入向量：

```bash
# 零宽字符（不可见指令）
grep -rPn '[\x{200B}-\x{200D}\x{FEFF}]' --include="*.md" --include="*.yaml" --include="*.json" . 2>/dev/null

# 包含指令的隐藏 HTML 注释
grep -rn '<!--' --include="*.md" . 2>/dev/null | grep -i 'ignore\|system\|admin\|instruction\|override\|forget'

# 注释中的 Base64（潜在隐藏载荷）
grep -rn -E '[#;].*[A-Za-z0-9+/]{20,}={0,2}' --include="*.py" --include="*.js" --include="*.ts" --include="*.md" \
  --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -10

# ANSI 转义序列
grep -rPn '\x1b\[|\x1b\]|\x1b\(' --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -10

# 空字节
grep -rPn '\x00' --exclude-dir={node_modules,vendor,.git,dist} . 2>/dev/null | head -5

# Markdown/配置中的嵌套命令执行
grep -rn -E '\$\([^)]+\)|`[^`]+`' --include="*.md" --include="*.yaml" --include="*.json" \
  --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -10
```

**评分：**
- 0 个注入向量 → +15 分
- 1-2 个（可能为误报）→ +10 分
- 3 个以上 → +5 分
- CLAUDE.md 中确认存在注入 → +0 分

---

### 阶段 4：依赖审计

针对项目运行相应的包审计：

```bash
# Node.js
[ -f "package-lock.json" ] && npm audit --json 2>/dev/null | jq '{total: .metadata.vulnerabilities.total, critical: .metadata.vulnerabilities.critical, high: .metadata.vulnerabilities.high}' 2>/dev/null

# Python
[ -f "requirements.txt" ] && pip-audit -r requirements.txt 2>/dev/null || [ -f "pyproject.toml" ] && pip-audit 2>/dev/null

# Rust
[ -f "Cargo.toml" ] && cargo audit 2>/dev/null

# Go
[ -f "go.mod" ] && govulncheck ./... 2>/dev/null
```

若未检测到包管理器，记录情况并跳过（不扣分）。

**评分：**
- 0 个漏洞 → +20 分
- 无严重/高危 → +15 分
- 1-3 个高危 → +10 分
- 存在严重漏洞 → +5 分
- 10 个以上高危或 3 个以上严重 → +0 分

---

### 阶段 5：钩子安全评估

验证 `guide/security-hardening.md` 中推荐的安全钩子是否已正确安装：

```bash
# 检查推荐的安全钩子
echo "=== Checking security hooks ==="

# PreToolUse 钩子（应拦截危险模式）
ls .claude/hooks/PreToolUse* 2>/dev/null || echo "⚠️ No PreToolUse hooks found"

# PostToolUse 钩子（应监控输出）
ls .claude/hooks/PostToolUse* 2>/dev/null || echo "⚠️ No PostToolUse hooks found"

# 检查是否存在提示词注入检测器
find . -path "*/hooks/*injection*" -o -path "*/hooks/*security*" -o -path "*/hooks/*scanner*" 2>/dev/null

# 检查 settings 中的钩子配置
grep -c "hooks" .claude/settings.json 2>/dev/null || echo "No hooks in settings.json"
```

**评分：**
- 已安装 PreToolUse 安全钩子 → +10 分
- 已安装 PostToolUse 输出扫描器 → +5 分
- 已安装提示词注入检测器钩子 → +5 分
- 完全未安装钩子 → +0 分

---

### 阶段 6：态势评分与报告

计算总分并生成报告。

**评分细则：**

| 类别 | 最高分 | 来源 |
|----------|-----------|--------|
| 配置安全（阶段 1） | 30 | /security-check 结果 |
| 密钥扫描（阶段 2） | 20 | 项目中发现的密钥 |
| 注入面（阶段 3） | 15 | 发现的注入向量 |
| 依赖项（阶段 4） | 20 | 漏洞审计 |
| 钩子安全（阶段 5） | 15 | 已安装的安全钩子 |
| **总计** | **100** | |

**阶段 1 评分明细：**
- 无严重（CRITICAL）问题 → +15 分
- 无高危（HIGH）问题 → +10 分
- 无中危（MEDIUM）问题 → +5 分
- 存在任意严重问题 → 该子项得 0 分

**等级标准：**

| 分数 | 等级 | 含义 |
|-------|-------|---------|
| 90-100 | A | 优秀 — 具备生产就绪的安全态势 |
| 75-89 | B | 良好 — 建议进行小幅改进 |
| 60-74 | C | 可接受 — 上线前解决高危问题 |
| 40-59 | D | 较差 — 存在严重安全缺口 |
| 0-39 | F | 危急 — 禁止部署，立即修复严重问题 |

## 输出格式

```
## 🛡️ 安全审计报告

**日期**：[时间戳]
**项目**：[目录名称]
**范围**：完整项目 + Claude Code 配置

### 安全态势评分：[XX]/100（等级 [X]）

[一句话评估]

### 各阶段结果

| 阶段 | 得分 | 满分 | 关键发现 |
|-------|-------|-----|-------------|
| 1. 配置安全 | XX | 30 | [摘要] |
| 2. 密钥扫描 | XX | 20 | [摘要] |
| 3. 注入面 | XX | 15 | [摘要] |
| 4. 依赖项 | XX | 20 | [摘要] |
| 5. 钩子安全 | XX | 15 | [摘要] |
| **总计** | **XX** | **100** | |

### 🔴 严重问题
[每项问题包含位置、描述及具体修复方法]

### 🟠 高危问题
[每项问题包含位置、描述及修复方法]

### 🟡 中危问题
[每项问题包含位置、描述及修复方法]

### 🔧 修复计划（按优先级排序）

| # | 操作 | 严重级别 | 工作量 | 命令/步骤 |
|---|--------|----------|--------|---------------|
| 1 | [操作] | 严重 | [时间] | [方法] |
| 2 | [操作] | 高危 | [时间] | [方法] |
| ... | | | | |

### 📊 基准对比

你的评分与 security-hardening.md 建议的对比：
- 已实现指南中的 [X] 项
- 缺失 [X] 项
- 下一步优先实现的 3 项：[...]

### 📚 参考资料
- 安全加固指南：guide/security-hardening.md
- 威胁数据库：examples/skills/update-threat-db/threat-db.yaml
- 快速检查：`/security-check`
- MCP 扫描工具：`npx mcp-scan`（Snyk）
```

$ARGUMENTS
