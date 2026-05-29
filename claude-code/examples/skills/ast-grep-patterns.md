> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: ast-grep-patterns
description: "教导 Claude 何时以及如何使用 ast-grep 进行结构化代码搜索的技能"
effort: low
---

# Claude Code 的 ast-grep 模式

> **技能**：教导 Claude 何时以及如何使用 ast-grep 进行结构化代码搜索

## 安装

```bash
# 安装 ast-grep 技能
npx skills add ast-grep/agent-skill

# 或通过插件市场
/plugin marketplace add
```

## 何时建议使用 ast-grep

在以下场景建议用户使用 ast-grep：

| 场景 | 信号词 | 为何使用 ast-grep？ |
|------|--------|-------------------|
| **大规模重构** | "重命名所有调用..."、"更改 API 签名"、">5万行" | 需要结构精度 |
| **框架迁移** | "React 17→18"、"Vue 2→3"、"升级依赖" | AST 感知转换 |
| **模式检测** | "找到没有...的函数"、"定位未使用的..."、"识别反模式" | 结构规则 |
| **代码库分析** | "哪些组件依赖于..."、"找到紧耦合的..." | 依赖关系图 |

**不建议使用的场景**：
- 简单字符串搜索（函数名、import）→ 使用 Grep
- 小型项目（<1万行）→ Grep 已足够
- 一次性搜索 → Grep 更快
- 语义搜索 → 使用 Serena MCP 或 grepai

## 决策树

```
用户请求分析：
├─ "查找字符串/文本" → Grep（原生）
├─ "按含义查找" → Serena MCP 或 grepai
├─ "按结构查找" → ast-grep（插件）
└─ 混合需求 → 从 Grep 开始，必要时升级
```

## 常用模式

### 1. 没有错误处理的异步函数

**用例**：查找缺少 try/catch 块的异步函数

```yaml
# ast-grep 规则
rule:
  pattern: |
    async function $FUNC($$$PARAMS) {
      $$$BODY
    }
  not:
    has:
      pattern: try { $$$TRY } catch
```

**使用时机**：安全审计、生产就绪检查

### 2. 使用特定钩子的 React 组件

**用例**：查找所有使用 `useEffect` 但没有清理函数的组件

```yaml
rule:
  pattern: |
    useEffect(() => {
      $$$BODY
    })
  not:
    has:
      pattern: return () => { $$$CLEANUP }
```

**使用时机**：内存泄漏检测、React 最佳实践审计

### 3. 超过参数阈值的函数

**用例**：查找参数超过5个的函数（复杂度异味）

```yaml
rule:
  pattern: function $NAME($P1, $P2, $P3, $P4, $P5, $P6, $$$REST) { $$$BODY }
```

**使用时机**：代码质量改进、重构候选项识别

### 4. 生产代码中的 Console.log

**用例**：从生产文件中移除调试日志

```yaml
rule:
  pattern: console.log($$$ARGS)
  inside:
    pattern: |
      class $CLASS {
        $$$METHODS
      }
```

**使用时机**：生产环境清理、发布前审计

### 5. 未使用的 React Props

**用例**：检测传入但从未使用的 props

```yaml
rule:
  pattern: |
    function $COMP({ $PROP, $$$OTHER }) {
      $$$BODY
    }
  not:
    has:
      pattern: $PROP
      inside: $$$BODY
```

**使用时机**：死代码消除、性能优化

### 6. 已废弃的 API 用法

**用例**：查找旧 API 方法的使用

```yaml
rule:
  any:
    - pattern: React.Component
    - pattern: componentWillMount
    - pattern: componentWillReceiveProps
```

**使用时机**：框架迁移、废弃清理

### 7. SQL 注入风险模式

**用例**：查找潜在的 SQL 注入漏洞

```yaml
rule:
  pattern: |
    db.query($TEMPLATE_LITERAL)
  where:
    $TEMPLATE_LITERAL:
      kind: template_string
```

**使用时机**：安全审计、漏洞扫描

### 8. 缺少 TypeScript 返回类型

**用例**：强制显式返回类型

```yaml
rule:
  pattern: |
    function $NAME($$$PARAMS) {
      $$$BODY
    }
  not:
    has:
      pattern: ': $TYPE'
```

**使用时机**：TypeScript 最佳实践、类型安全改进

### 9. 大型 Switch 语句（重构候选项）

**用例**：查找超过10个分支的 switch 语句

```yaml
rule:
  pattern: |
    switch ($EXPR) {
      $C1: $$$B1
      $C2: $$$B2
      $C3: $$$B3
      $C4: $$$B4
      $C5: $$$B5
      $C6: $$$B6
      $C7: $$$B7
      $C8: $$$B8
      $C9: $$$B9
      $C10: $$$B10
      $C11: $$$B11
    }
```

**使用时机**：降低复杂度、多态重构

### 10. 空 Catch 块（被吞掉的错误）

**用例**：查找静默失败的错误处理

```yaml
rule:
  pattern: |
    try {
      $$$TRY
    } catch ($ERR) {
      // 空或仅有注释
    }
```

**使用时机**：调试神秘故障、错误处理审计

## 配置复杂度与价值

| 代码库规模 | 值得配置？ | 替代方案 |
|-----------|-----------|---------|
| <1万行 | ❌ 不 | 使用 Grep |
| 1万-5万行 | ⚠️ 也许 | 先用 Grep，必要时升级 |
| 5万-20万行 | ✅ 是 | ast-grep 处理结构，Grep 处理文本 |
| >20万行 | ✅ 绝对是 | ast-grep + Serena MCP 组合 |

## 故障排查

### ast-grep 未找到

```bash
# 验证安装
npx ast-grep --version

# 重新安装技能
npx skills add ast-grep/agent-skill --force
```

### Claude 未使用 ast-grep

**问题**：Claude 使用 Grep 而非 ast-grep

**解决方案**：在请求中明确说明
- ❌ "查找异步函数"
- ✅ "使用 ast-grep 查找异步函数"

### 性能问题

**问题**：ast-grep 在大型代码库上速度慢

**解决方案**：
1. 缩小搜索范围：`ast-grep --path src/components/`
2. 使用文件过滤：`ast-grep --lang tsx`
3. 缓存结果供迭代优化

### 模式不匹配

**问题**：ast-grep 模式与预期代码不匹配

**调试步骤**：
1. 单独测试模式：`ast-grep -p 'your-pattern' file.js`
2. 检查 AST 结构：`ast-grep --debug-query`
3. 逐步简化模式
4. 验证语言语法（JS vs TS vs JSX）

## 集成示例

### 工作流：使用 ast-grep 的提交前钩子

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 检查暂存文件中的 console.log
if ast-grep -p 'console.log($$$)' $(git diff --cached --name-only); then
  echo "❌ 发现 console.log 语句"
  exit 1
fi
```

### 工作流：迁移脚本

```bash
#!/bin/bash
# 将 React 类组件迁移至钩子

# 查找所有类组件
ast-grep -p 'class $C extends React.Component' --json > components.json

# 处理每个组件
jq -r '.[] | .file' components.json | while read file; do
  echo "正在迁移：$file"
  # ... 转换逻辑
done
```

### 工作流：安全审计

```bash
#!/bin/bash
# security-audit.sh

echo "=== 安全审计 ==="

# SQL 注入风险
ast-grep -p 'db.query(`${$VAR}`)' --lang ts

# XSS 风险
ast-grep -p 'innerHTML = $VAR' --lang js

# 硬编码密钥
ast-grep -p 'password: "$PASSWORD"' --lang ts
```

## Claude 提示词模板

### 模板1：大规模重构

```
我需要在我们的代码库（约 [规模] 行）中重构 [功能]。

使用 ast-grep：
1. 查找 [旧模式] 的所有实例
2. 确定哪些文件会受到影响
3. 建议转换策略
4. 制定分阶段迁移计划

仅进行分析，等待我批准后再进行更改。
```

### 模板2：框架迁移

```
我们正在从 [旧框架 v1] 迁移到 [新框架 v2]。

使用 ast-grep：
1. 查找所有已废弃 API 的使用
2. 映射到新 API 等效项
3. 估算迁移工作量（受影响文件数）
4. 识别高风险更改

提供显示迁移顺序的依赖关系图。
```

### 模板3：代码质量审计

```
使用 ast-grep 对 [目录] 进行代码质量审计。

重点关注：
- 参数超过5个的函数
- 没有错误处理的异步函数
- 空 catch 块
- 未使用的函数参数

按严重程度对问题排序并提供重构建议。
```

## 进阶：将 ast-grep 与其他工具结合

### ast-grep + Serena MCP

```bash
# 1. ast-grep 查找结构模式
ast-grep -p 'async function $F' --json > async-funcs.json

# 2. Serena 查找符号和依赖
claude mcp call serena find_symbol --name "authenticate"

# 3. 结合洞察获取完整上下文
# ast-grep："这是一个异步函数"
# Serena："被其他12个函数调用"
```

### ast-grep + grepai

```bash
# 1. grepai 用于语义搜索
# "查找与认证相关的代码"

# 2. ast-grep 用于结构细化
# "在这些结果中，哪些是没有错误处理的异步函数？"
```

## 最佳实践

1. **从简单开始**：先用 Grep，必要时升级到 ast-grep
2. **测试模式**：在整个代码库运行前先在小文件上验证
3. **文档化模式**：将成功的模式保存为可复用规则
4. **明确请求**：始终明确告诉 Claude 何时使用 ast-grep
5. **组合工具**：ast-grep 处理结构，Grep 处理文本，Serena 处理符号

## 资源

- [ast-grep 文档](https://ast-grep.github.io/)
- [模式演练场](https://ast-grep.github.io/playground.html)
- [ast-grep GitHub](https://github.com/ast-grep/ast-grep)
- [Claude 技能](https://github.com/ast-grep/claude-skill)

---

**最后更新**：2026年1月
**兼容版本**：Claude Code 2.1.7+
