> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: explain
description: 以可调节的深度级别解释代码、概念或系统行为
argument-hint: <file_or_concept>
effort: low
---

# 代码解释器

以可调节的深度级别解释代码、概念或系统行为。

## 用途

获取清晰的解释，涵盖：
- 特定代码的工作原理
- 某些模式的使用原因
- 某个系统/模块的功能
- 架构决策与权衡

## 操作说明

### 第一步：确定范围

明确需要解释的内容：
- **文件**：整个文件的结构与用途
- **函数/方法**：具体的实现细节
- **概念**：架构模式或设计决策
- **流程**：数据/控制流在系统中的流转方式

### 第二步：评估复杂度

```
简单（阅读时间 1-2 分钟）   → 快速摘要，只列关键点
标准（阅读时间 3-5 分钟）   → 用途、工作原理、关键决策
深度（阅读时间 10+ 分钟）   → 完整分解、备选方案、权衡分析
```

### 第三步：收集上下文

```bash
# 文件解释
head -50 "$FILE"  # 查看导入项和结构

# 函数解释
grep -A 30 "function $NAME\|def $NAME\|fn $NAME" "$FILE"

# 模块解释
ls -la "$DIR"
cat "$DIR/index.ts" 2>/dev/null || cat "$DIR/__init__.py" 2>/dev/null
```

### 第四步：组织解释结构

## 输出格式

---

### 📖 解释：[目标]

**范围**：[文件/函数/概念/流程]
**深度**：[简单/标准/深度]

### 它做什么

[1-3 句话描述其用途]

### 它如何工作

[根据深度级别逐步分解]

### 关键决策

| 决策 | 原因 | 替代方案 |
|----------|-----|-------------|
| [所作选择] | [理由] | [其他可行方案] |

### 使用示例

```typescript
// 如何正确使用
```

### 相关代码

- `path/to/related.ts` - [关联关系]
- `path/to/dependency.ts` - [关联关系]

### 💡 学习笔记（使用 --learn 参数时）

[理解更广泛模式的补充上下文]

---

## 深度级别

### 简单（`/explain --simple`）

```markdown
**validateUser()** 检查用户对象是否包含必填字段
（email、password）并返回布尔值。使用正则表达式验证邮箱格式。
```

### 标准（`/explain` - 默认）

```markdown
**validateUser(user: User): ValidationResult**

**用途**：在数据库操作之前验证用户输入。

**流程**：
1. 检查必填字段是否存在（email、password）
2. 用正则表达式验证邮箱格式
3. 检查密码是否符合要求（8 位以上，含特殊字符）
4. 返回 { valid: boolean, errors: string[] }

**调用方**：signup()、updateProfile()
```

### 深度（`/explain --deep`）

```markdown
[标准内容基础上，另加：]

**设计决策**：
- 返回 ValidationResult 而非抛出异常，以支持批量验证
- 选用正则而非第三方库，以实现零依赖
- 密码规则通过 config.ts 可配置

**权衡分析**：
- 优点：速度快，无依赖
- 缺点：正则邮箱验证不完全符合 RFC 规范

**备选方案**：
- Zod schema：功能更强，但增加 50KB 体积
- Class-validator：更适合装饰器模式，但面向对象较重
```

## 使用示例

**解释一个文件：**
```
/explain src/auth/middleware.ts
```

**解释一个函数：**
```
/explain payments.ts 中的 handleWebhook 函数
```

**解释一个概念：**
```
/explain 我们的事件溯源是如何工作的
```

**指定深度解释：**
```
/explain --deep 认证流程
/explain --simple useCallback 的作用
```

**用于学习的解释：**
```
/explain --learn 这里使用的 repository 模式
```

## 使用技巧

1. **具体明确**："解释第 45-60 行" 优于 "解释这个文件"
2. **说明水平**："我是 TypeScript 新手" 有助于校准解释深度
3. **追问细节**："为什么不用 X？" 能加深理解
4. **要求类比**："用熟悉 Python 但不懂 TS 的视角来解释"

$ARGUMENTS
