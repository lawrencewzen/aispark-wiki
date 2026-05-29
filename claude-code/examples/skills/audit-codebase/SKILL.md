> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: audit-codebase
description: 代码库健康度审计，对 7 个维度评分并提供改进路线图
argument-hint: "[path] [--focus security|performance|quality]"
effort: medium
disable-model-invocation: true
---

# 代码库健康度审计

对代码库的 7 个健康维度进行评分，识别薄弱环节，并给出优先级改进路线图。每个维度以 1-10 分评分，附带具体可操作的发现项。

**耗时**：3-8 分钟（取决于代码库规模）| **范围**：完整项目

## 说明

你是一名高级工程顾问，正在执行代码库健康评估。分析项目的全部 7 个维度（或 `$ARGUMENTS` 指定的子集），对每个维度评分，并生成改进路线图。

若 `$ARGUMENTS` 包含维度名称（如 "secrets security tests"），仅审计这些维度；否则审计全部 7 个。

---

### 维度 1：密钥（权重：15%）

扫描代码中硬编码的凭据、API 密钥和敏感数据。

```bash
# 代码中的 API 密钥和令牌
grep -rn --include="*.{js,ts,py,go,java,rb,php,yaml,yml,json,toml,env,cfg,ini,conf}" \
  -E '(?i)(api[_-]?key|apikey|secret[_-]?key|password|passwd|token|bearer)\s*[=:]\s*["'\''"][^"'\'']{8,}' \
  --exclude-dir={node_modules,vendor,.git,dist,build,target,__pycache__,.venv} . 2>/dev/null | head -20

# 已知提供商的密钥模式
grep -rn -E 'sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36}|AKIA[A-Z0-9]{16}|xox[bps]-[a-zA-Z0-9\-]{20,}' \
  --exclude-dir={node_modules,vendor,.git,dist,build,target} . 2>/dev/null | head -10

# 已提交的 .env 文件
find . -name ".env*" -not -name ".env.example" -not -path "*/node_modules/*" -not -path "*/.git/*" -type f 2>/dev/null

# .gitignore 覆盖情况
[ -f ".gitignore" ] && {
  for pattern in ".env" "*.pem" "*.key" "*.p12"; do
    grep -q "$pattern" .gitignore 2>/dev/null && echo "OK: $pattern in .gitignore" || echo "MISSING: $pattern not in .gitignore"
  done
}
```

**评分标准：**
- 10 分：零密钥，.gitignore 覆盖所有敏感模式，存在 .env.example
- 7-9 分：代码中无密钥，.gitignore 存在轻微缺口
- 4-6 分：发现 1-3 处潜在密钥（可能是误报），或 .env 已提交
- 1-3 分：代码中存在多处密钥，私钥已提交，无 .gitignore 保护

---

### 维度 2：安全性（权重：15%）

检查 OWASP 类漏洞和不安全模式。

```bash
# SQL 注入模式
grep -rn --include="*.{js,ts,py,java,go,rb,php}" \
  -E '(query|execute|exec)\s*\(\s*[`"'\''"].*\+|\$\{|%s|\.format\(' \
  --exclude-dir={node_modules,vendor,.git,dist,build,target,test,__test__} . 2>/dev/null | head -15

# eval/exec 使用情况
grep -rn -E '\b(eval|exec|execSync|Function\(|setTimeout\([^,]*[+`]|setInterval\([^,]*[+`])' \
  --include="*.{js,ts,py}" --exclude-dir={node_modules,vendor,.git,dist} . 2>/dev/null | head -10

# 不安全的反序列化
grep -rn -E '(pickle\.loads|yaml\.load\(|JSON\.parse\(.*user|unserialize\()' \
  --exclude-dir={node_modules,vendor,.git,dist} . 2>/dev/null | head -10

# 路由/端点缺少输入验证
grep -rn -E '(app\.(get|post|put|delete|patch)|router\.(get|post|put|delete))' \
  --include="*.{js,ts}" --exclude-dir={node_modules,.git,dist} . 2>/dev/null | wc -l
```

**评分标准：**
- 10 分：无注入模式，无 eval/exec，所有端点有输入验证，配置了 CSP 头
- 7-9 分：轻微问题（非用户暴露代码中有 1-2 处 eval 使用）
- 4-6 分：存在注入模式，多个端点缺少验证
- 1-3 分：存在活跃 SQL 注入风险，eval 处理用户输入，无输入消毒处理

---

### 维度 3：依赖项（权重：15%）

审计包健康状况、已知 CVE 和版本新鲜度。

```bash
# Node.js 审计
[ -f "package-lock.json" ] && npm audit --json 2>/dev/null | jq '.metadata.vulnerabilities' 2>/dev/null
[ -f "package.json" ] && npx npm-check 2>/dev/null | tail -20

# Python
[ -f "requirements.txt" ] && pip-audit -r requirements.txt 2>/dev/null | tail -20
[ -f "pyproject.toml" ] && pip-audit 2>/dev/null | tail -20

# Rust
[ -f "Cargo.toml" ] && cargo audit 2>/dev/null | tail -20

# Go
[ -f "go.mod" ] && govulncheck ./... 2>/dev/null | tail -20

# 锁文件检查
for lockfile in package-lock.json yarn.lock pnpm-lock.yaml Cargo.lock go.sum poetry.lock; do
  [ -f "$lockfile" ] && echo "OK: $lockfile exists"
done
[ ! -f "package-lock.json" ] && [ ! -f "yarn.lock" ] && [ ! -f "pnpm-lock.yaml" ] && [ -f "package.json" ] && echo "MISSING: No lockfile for Node.js project"
```

**评分标准：**
- 10 分：零 CVE，存在锁文件，所有依赖项不超过 6 个月
- 7-9 分：无严重/高危 CVE，少量过时包
- 4-6 分：1-3 个高危 CVE，或超过 50% 的依赖项已超过一年未更新
- 1-3 分：存在严重 CVE，无锁文件，依赖项已废弃

---

### 维度 4：结构（权重：10%）

评估文件组织、命名规范和模块边界。

```bash
# 每个顶层目录的文件数
for dir in */; do
  [ -d "$dir" ] && [ "$dir" != "node_modules/" ] && [ "$dir" != ".git/" ] && [ "$dir" != "vendor/" ] && \
    echo "$dir: $(find "$dir" -type f -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | wc -l) files"
done

# 深度嵌套文件（复杂度指标）
find . -type f -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -mindepth 6 2>/dev/null | head -10

# 混用命名规范
find . -type f -name "*_*" -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | head -5
find . -type f -name "*-*" -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | head -5

# 循环依赖指标（JS/TS 项目）
[ -f "package.json" ] && npx madge --circular --extensions ts,js src/ 2>/dev/null | head -20
```

**评分标准：**
- 10 分：模块边界清晰，命名统一，无循环依赖，层级扁平
- 7-9 分：结构良好，存在轻微不一致
- 4-6 分：规范混用，存在循环依赖，模块边界不清晰
- 1-3 分：无清晰结构，文件嵌套过深，循环依赖泛滥

---

### 维度 5：测试（权重：15%）

评估测试覆盖率、测试质量和测试实践。

```bash
# 测试文件数 vs 源文件数
TEST_COUNT=$(find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" -o -path "*/test/*" -o -path "*/__tests__/*" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | wc -l)
SRC_COUNT=$(find . -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.go" -o -name "*.java" \) \
  -not -name "*.test.*" -not -name "*.spec.*" -not -name "test_*" \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/dist/*" 2>/dev/null | wc -l)
echo "Test files: $TEST_COUNT | Source files: $SRC_COUNT | Ratio: $(echo "scale=2; $TEST_COUNT / ($SRC_COUNT + 1)" | bc)"

# 覆盖率配置检查
for cfg in jest.config.* vitest.config.* .nycrc .coveragerc pytest.ini setup.cfg; do
  [ -f "$cfg" ] && echo "OK: $cfg exists"
done

# 覆盖率报告（如有）
[ -d "coverage" ] && [ -f "coverage/coverage-summary.json" ] && cat coverage/coverage-summary.json | jq '.total' 2>/dev/null

# 快照测试数量（潜在维护负担）
find . -name "*.snap" -not -path "*/node_modules/*" 2>/dev/null | wc -l
```

**评分标准：**
- 10 分：测试比例 >0.8，覆盖率 >80%，CI 运行测试，无过时快照
- 7-9 分：测试比例 >0.5，覆盖率 >60%，存在覆盖率配置
- 4-6 分：存在部分测试但缺口明显，无覆盖率跟踪
- 1-3 分：测试比例 <0.2 或完全无测试

---

### 维度 6：导入（权重：10%）

检查未使用的导入、循环依赖和类型覆盖情况。

```bash
# 未使用的导入（TypeScript/JavaScript）
[ -f "tsconfig.json" ] && npx tsc --noEmit 2>&1 | grep -c "declared but" 2>/dev/null
[ -f "tsconfig.json" ] && npx tsc --noEmit 2>&1 | grep "declared but" | head -10

# TypeScript 严格模式
[ -f "tsconfig.json" ] && grep -E '"strict"|"noImplicitAny"|"strictNullChecks"' tsconfig.json 2>/dev/null

# Python 未使用导入
[ -f "pyproject.toml" ] || [ -f "setup.py" ] && python -m pyflakes . 2>/dev/null | grep "imported but unused" | head -10

# 通配符导入（代码异味）
grep -rn 'import \*' --include="*.{py,ts,js}" --exclude-dir={node_modules,vendor,.git} . 2>/dev/null | head -10
```

**评分标准：**
- 10 分：零未使用导入，启用 TypeScript 严格模式，无通配符导入
- 7-9 分：少于 5 个未使用导入，已启用严格模式但存在轻微缺口
- 4-6 分：5-20 个未使用导入，无严格模式，存在通配符导入
- 1-3 分：超过 20 个未使用导入，通配符导入泛滥，无类型检查

---

### 维度 7：AI 模式（权重：20%）

评估 Claude Code 配置成熟度和 AI 辅助开发就绪程度。

```bash
# CLAUDE.md 是否存在及质量
[ -f "CLAUDE.md" ] && echo "OK: CLAUDE.md exists ($(wc -l < CLAUDE.md) lines)" || echo "MISSING: No CLAUDE.md"
[ -f ".claude/settings.json" ] && echo "OK: .claude/settings.json exists" || echo "MISSING: No .claude/settings.json"

# 自定义命令
COMMANDS=$(find .claude/commands -name "*.md" 2>/dev/null | wc -l)
echo "Custom commands: $COMMANDS"

# 钩子
HOOKS_CFG=$(grep -c "hooks" .claude/settings.json 2>/dev/null || echo "0")
echo "Hook configurations: $HOOKS_CFG"

# 规则文件
RULES=$(find .claude/rules -name "*.md" 2>/dev/null | wc -l)
echo "Rule files: $RULES"

# 智能体
AGENTS=$(find .claude/agents -name "*.md" 2>/dev/null | wc -l)
echo "Agent definitions: $AGENTS"

# 技能
SKILLS=$(find .claude/skills -name "*.md" 2>/dev/null | wc -l)
echo "Skills: $SKILLS"

# AI 产物的 .gitignore 配置
grep -q "claude" .gitignore 2>/dev/null && echo "OK: Claude patterns in .gitignore" || echo "INFO: No Claude patterns in .gitignore"
```

**评分标准：**
- 10 分：CLAUDE.md 含规范说明，已配置钩子、自定义命令、规则、智能体
- 7-9 分：CLAUDE.md 存在且含项目上下文，有部分命令或规则
- 4-6 分：基础 CLAUDE.md，无钩子或命令
- 1-3 分：无 CLAUDE.md 或 CLAUDE.md 为空

---

## 评分与报告

### 综合评分计算

```
综合分 = (密钥 * 0.15) + (安全性 * 0.15) + (依赖项 * 0.15) +
         (结构 * 0.10) + (测试 * 0.15) + (导入 * 0.10) +
         (AI 模式 * 0.20)
```

保留一位小数。

### 输出格式

```markdown
## 代码库健康度审计

**项目**：[目录名称]
**日期**：[时间戳]
**已审计维度**：[全部 7 个或过滤后的子集]

### 综合评分：[X.X] / 10

| 维度 | 评分 | 权重 | 加权分 | 关键发现 |
|------|------|------|--------|---------|
| 密钥 | X/10 | 15% | X.XX | [一句话摘要] |
| 安全性 | X/10 | 15% | X.XX | [一句话摘要] |
| 依赖项 | X/10 | 15% | X.XX | [一句话摘要] |
| 结构 | X/10 | 10% | X.XX | [一句话摘要] |
| 测试 | X/10 | 15% | X.XX | [一句话摘要] |
| 导入 | X/10 | 10% | X.XX | [一句话摘要] |
| AI 模式 | X/10 | 20% | X.XX | [一句话摘要] |
| **综合** | | **100%** | **X.XX** | |

### 详细发现

#### 🔴 严重（立即修复）
- [发现项，附文件:行号引用及具体修复方案]

#### 🟡 警告（本周修复）
- [发现项，附背景说明和建议方案]

#### 🟢 信息（可选改进）
- [观察项，附可选建议]

### 改进路线图

[根据综合评分显示对应层级]

#### 层级 1：基础（当前评分 <5，目标：5）
优先消除严重风险，其他事项暂缓。

| 优先级 | 行动项 | 维度 | 影响 | 工作量 |
|--------|--------|------|------|--------|
| 1 | [具体行动] | [维度] | [分数提升] | [时间估算] |
| 2 | [具体行动] | [维度] | [分数提升] | [时间估算] |
| ... | | | | |

#### 层级 2：稳固（当前评分 5-7，目标：8）
在基础之上建立可靠的工程实践。

| 优先级 | 行动项 | 维度 | 影响 | 工作量 |
|--------|--------|------|------|--------|
| 1 | [具体行动] | [维度] | [分数提升] | [时间估算] |
| ... | | | | |

#### 层级 3：卓越（当前评分 8+，目标：10）
精益求精，最大化团队开发效率。

| 优先级 | 行动项 | 维度 | 影响 | 工作量 |
|--------|--------|------|------|--------|
| 1 | [具体行动] | [维度] | [分数提升] | [时间估算] |
| ... | | | | |

### 速效改进（每项 < 30 分钟）
1. [以最小投入提升评分的行动项]
2. [...]
3. [...]
```

### 严重程度分布

约 70% 的发现项应可自动化处理（脚本、linter、CI 检查均可检测）。将其余 30% 标记为需要人工判断，并说明自动化无法覆盖的原因。

---

**参考来源**：
- Variant Systems 代码库分析插件（variantsystems.io，2026 年 2 月）：7 维度分析框架
- OWASP Top 10（2021 年版）：安全性维度模式
- Claude Code 安全加固指南：AI 模式维度基线

$ARGUMENTS
