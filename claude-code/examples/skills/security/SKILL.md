> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: security
description: 针对 OWASP Top 10 漏洞的快速安全评估
argument-hint: "[path] [--depth quick|full]"
effort: medium
disable-model-invocation: true
---

# 安全快速审计

针对 OWASP Top 10 漏洞的快速安全评估。

## 目的

执行快速安全扫描以识别常见漏洞：
- 硬编码的密钥和凭证
- SQL 注入风险
- XSS 漏洞
- 不安全的依赖
- 认证/授权问题

## 操作说明

### 第1步：密钥扫描

```bash
# Common secret patterns
grep -rn --include="*.{js,ts,py,go,java,rb,php,env}" \
  -E "(password|secret|api_key|apikey|token|auth|credential).*[=:].*['\"][^'\"]{8,}['\"]" \
  --exclude-dir={node_modules,vendor,.git,dist,build} . 2>/dev/null | head -20

# .env files that might be committed
find . -name ".env*" -not -path "*/node_modules/*" -type f 2>/dev/null

# Check if secrets are gitignored
[ -f ".gitignore" ] && grep -q "\.env" .gitignore && echo "✅ .env in .gitignore" || echo "⚠️ .env NOT in .gitignore"
```

### 第2步：注入漏洞

```bash
# SQL injection patterns (raw queries with string concat)
grep -rn --include="*.{js,ts,py,go,java,php}" \
  -E "(query|execute|raw|sql).*\+.*\$|f['\"].*SELECT|\.format\(.*SELECT" \
  --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -15

# Command injection patterns
grep -rn --include="*.{js,ts,py,go,rb,php}" \
  -E "(exec|spawn|system|shell_exec|popen)\s*\(" \
  --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -15
```

### 第3步：XSS 模式

```bash
# Dangerous innerHTML/dangerouslySetInnerHTML usage
grep -rn --include="*.{js,ts,jsx,tsx,vue}" \
  -E "(innerHTML|dangerouslySetInnerHTML|v-html)" \
  --exclude-dir={node_modules,.git,dist} . 2>/dev/null | head -15

# Unescaped template literals in HTML context
grep -rn --include="*.{js,ts,jsx,tsx}" \
  -E "\`.*\$\{.*\}.*<" \
  --exclude-dir={node_modules,.git,dist} . 2>/dev/null | head -10
```

### 第4步：依赖检查

```bash
# Check for known vulnerabilities in npm packages
[ -f "package-lock.json" ] && npm audit --json 2>/dev/null | jq '{vulnerabilities: .metadata.vulnerabilities}' 2>/dev/null

# Check for outdated packages with security issues
[ -f "package.json" ] && npm outdated --json 2>/dev/null | jq 'to_entries | map(select(.value.current != .value.latest)) | length' 2>/dev/null
```

### 第5步：认证与会话问题

```bash
# Hardcoded JWT secrets
grep -rn --include="*.{js,ts,py,go}" \
  -E "(jwt|JWT).*secret.*[=:].*['\"].{8,}['\"]" \
  --exclude-dir={node_modules,vendor,.git} . 2>/dev/null

# Missing CSRF protection patterns
grep -rn --include="*.{js,ts,py}" \
  -E "(POST|PUT|DELETE|PATCH).*fetch|axios\.(post|put|delete|patch)" \
  --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -10
```

## 输出格式

---

### 🛡️ 安全审计报告

**扫描日期**：[时间戳]
**扫描范围**：[扫描的目录]

### 🔴 严重问题

| 问题 | 位置 | 描述 |
|------|------|------|
| [类型] | [文件:行号] | [简要描述] |

### 🟠 高危

| 问题 | 位置 | 建议 |
|------|------|------|
| [类型] | [文件:行号] | [修复建议] |

### 🟡 中危

| 问题 | 位置 | 备注 |
|------|------|------|
| [类型] | [文件:行号] | [上下文] |

### 📊 摘要

- **严重**：X 个问题
- **高危**：X 个问题
- **中危**：X 个问题
- **依赖**：X 个漏洞

### 🔧 快速修复

1. [最高优先级修复，附命令/代码]
2. [第二优先级]
3. [第三优先级]

---

## 严重性级别

| 级别 | 示例 | 行动 |
|------|------|------|
| 🔴 严重 | 硬编码生产密钥、SQL 注入 | 立即修复 |
| 🟠 高危 | 缺少认证、XSS 向量 | 部署前修复 |
| 🟡 中危 | 过时依赖、缺少 CSRF 防护 | 计划修复 |
| 🟢 低危 | 最佳实践违规 | 跟踪改进 |

## 用法

**完整审计：**
```
/security
```

**聚焦特定领域：**
```
/security auth
/security deps
/security injection
```

**特定文件/目录：**
```
/security src/api/
```

## 注意事项

- 这是快速启发式扫描，不是全面的安全审计
- 生产系统请配合专用工具使用（Snyk、SonarQube、OWASP ZAP）
- 可能存在误报——请手动验证发现的问题
- 参见 `examples/hooks/security-hooks.sh` 了解自动化预提交安全检查

$ARGUMENTS
