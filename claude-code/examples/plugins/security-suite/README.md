> 📚 **AI Spark Wiki** · Claude Code 知识库

# 安全套件插件

**5 分钟内完成 Claude Code 工作流的全面安全加固。**

本插件集成了保护 Claude Code 配置所需的一切：自动化安全扫描、提交前拦截门禁、威胁追踪、合规检查及配置审计。

## 包含内容

✓ **安全审计智能体** — 威胁建模与 CVE 分析专家
✓ **快速安全检查** — 30 秒配置扫描
✓ **完整安全审计** — 6 阶段深度检测，满分 100
✓ **提交前安全门禁** — 在执行前拦截危险操作
✓ **配置审计** — 验证智能体、技能、命令的质量
✓ **合规模板** — 自动化治理清单

## 快速安装

```bash
# 在此目录下执行：
bash install.sh

# 验证安装：
/security-check
/security-audit
/audit-agents-skills
```

**耗时：** 5 分钟。无外部依赖。

## 组件说明

### 1. 安全审计智能体

专注于安全分析的专家型智能体。

```bash
# 分配复杂威胁建模任务：
@security-auditor Analyze this multi-agent setup for supply chain risks
```

**能力：**
- CVE 追踪与影响评估
- 供应链漏洞检测
- MCP 服务器安全审查
- 自定义技能/钩子风险分析
- 合规要求映射

### 2. 快速安全检查（`/security-check`）

30 秒扫描。在提交或部署前运行。

```bash
/security-check
```

**检查项：**
- ✓ CLAUDE.md 隔离（无密钥泄露）
- ✓ MCP 服务器配置（是否存在危险权限？）
- ✓ 钩子执行范围（是否过于宽松？）
- ✓ 智能体工具限制（是否已正确沙盒化？）
- ✓ Settings.json 权限（是否已锁定？）

**输出示例：**
```
Security Check: 87/100 ✓ PASS
- Agents properly isolated ✓
- No dangerous MCP configs ⚠️ REVIEW: 1 issue
- Hooks validated ✓
- CLAUDE.md secure ✓
```

### 3. 完整安全审计（`/security-audit`）

6 阶段深度检测，耗时 2-5 分钟，提供全面威胁评估。

```bash
/security-audit
```

**6 个阶段：**
1. **配置审计** — 设置、权限、密钥
2. **智能体安全** — 工具限制、隔离、能力
3. **技能/钩子分析** — 代码质量、危险模式
4. **MCP 审查** — 服务器权限、令牌处理
5. **供应链** — 依赖分析、外部集成
6. **合规检查** — 企业要求、治理规范

**输出示例：**
```
Security Audit Report
════════════════════════════════════════════
Config Audit:      ████░░░░░░  80/100
Agent Security:    ██████░░░░  85/100
Skill/Hook Review: ██░░░░░░░░  20/100  ⚠️ ACTION REQUIRED
MCP Vetting:       ███░░░░░░░  65/100  ⚠️ WARNINGS
Supply Chain:      ████░░░░░░  78/100
Compliance:        ██████░░░░  88/100

OVERALL: 72/100 — Intermediate Security Posture

Critical Issues (fix immediately):
  • 3 skills use eval() — Replace with safe parsing
  • GitHub MCP has write access to all repos — Scope to specific repos
  • Hook security-check.sh runs as root — Drop privileges

Warnings (fix within 2 weeks):
  • CLAUDE.md stores test API keys — Use environment variables
  • Agent tool permissions not clearly documented

Recommendations:
  • Read: guide/security/security-hardening/
  • Reference: guide/security/ for threat patterns
  • Use: /self-assessment → identify security gaps → take training
```

### 4. 配置审计（`/audit-agents-skills`）

验证自定义智能体、技能、命令的质量。

```bash
/audit-agents-skills
/audit-agents-skills --fix          # 获取修复建议
/audit-agents-skills ~/other-dir    # 审计其他项目
```

**检查项：**
- ✓ 智能体具备适当的工具限制
- ✓ 技能使用正确的 frontmatter 结构
- ✓ 命令具有清晰的描述
- ✓ 无硬编码密钥或 API 密钥
- ✓ 所有工具调用安全

### 5. 安全钩子

**提交前钩子：** `security-gate.sh`
在执行前拦截危险操作：
- 阻止自定义代码中的 `eval()`
- 阻止密钥泄露（API 密钥、令牌）
- 验证钩子语法
- 检查命令注入模式

**执行后钩子：** `security-check.sh`
对输出进行安全问题验证：
- 检测意外的密钥泄露
- 检查路径遍历尝试
- 监控资源使用（DoS 检测）
- 标记危险模式

## 使用场景

### 场景 1：加固团队配置

```
1. 运行：/security-audit
2. 审查报告并修复严重问题
3. 运行：/audit-agents-skills --fix
4. 与团队共享结果
5. 每月重新审计以跟踪改进进度
```

### 场景 2：评估第三方技能

```
1. 将技能下载到 examples/skills/
2. 运行：/audit-agents-skills
3. 审查工具限制和代码模式
4. 安全则集成，否则拒绝
```

### 场景 3：生产环境加固

```
1. 运行：/security-audit（建立当前安全基线）
2. 修复所有严重问题
3. 阅读：guide/security/production-safety/
4. 实施推荐模式
5. 重新审计直至得分达到 90+
```

### 场景 4：合规验证

```
1. 运行：/security-audit
2. 检查合规阶段（第 6 项）
3. 将差距映射到监管要求
4. 创建 compliance.md 记录合规状况
5. 每季度审计一次
```

## 配置

### 启用钩子

如果钩子未自动启用，请添加到 `.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/bash/security-gate.sh",
            "timeout": 3000
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/bash/security-check.sh",
            "async": true
          }
        ]
      }
    ]
  }
}
```

### 环境变量

配置安全参数：

```bash
# 最大工具超时时间（防止 DoS）
export CLAUDE_SECURITY_TIMEOUT=30000

# 启用严格模式（阻止所有未审查代码）
export CLAUDE_SECURITY_STRICT=true

# 安全审计级别（quick|standard|paranoid）
export CLAUDE_SECURITY_LEVEL=standard
```

## 学习路径

**安装后：**

1. **了解威胁** → 阅读 `guide/security/security-hardening/`
2. **评估配置** → 运行 `/security-check`
3. **获取详细报告** → 运行 `/security-audit`
4. **修复问题** → 按照建议操作
5. **深入学习** → 使用 `/self-assessment` 识别安全知识盲区
6. **验证** → 重新审计直至达到目标分数（推荐 85+/100）

## 卸载

```bash
bash uninstall.sh
```

此操作会移除所有安全套件组件，但保留备份文件（`.bak` 文件）。

## 获取帮助

- **有疑问？** → 查看 `guide/security/` 了解威胁模式和缓解措施
- **具体审计问题？** → 使用安全审计智能体
- **需要威胁情报？** → 查看 `guide/core/known-issues/` 获取活跃 CVE
- **团队推广？** → 在团队中运行审计，汇总结果，识别共同短板

## 版本信息

- **插件版本：** 1.0.0
- **要求：** Claude Code 2.1.0+
- **兼容模型：** Opus 4.7、Sonnet 4.6、Haiku 4.5

## 许可证

CC BY-SA 4.0 — 自由使用，按需修改，共享改进成果。

---

**准备好加固你的配置了吗？** 现在运行 `/security-check`。
