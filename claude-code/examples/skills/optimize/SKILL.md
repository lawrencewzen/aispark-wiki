> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: optimize
description: 分析并为代码、查询或系统提供性能优化建议
argument-hint: "<file_or_module> [--focus speed|memory|bundle]"
effort: medium
disable-model-invocation: true
---

# 性能优化器

分析并为代码、查询或系统提供性能优化建议。

## 目标

识别优化机会：
- 运行时性能瓶颈
- 内存使用问题
- 数据库查询低效
- Bundle 体积问题
- 算法复杂度

## 说明

### 第一步：确定范围

确定优化目标：
- **函数**：单函数性能
- **模块**：相关函数/类
- **查询**：数据库查询优化
- **Bundle**：前端 bundle 分析
- **系统**：架构级别优化

### 第二步：性能分析

#### 运行时分析

```bash
# Find potentially slow patterns
grep -rn "forEach\|\.map\|\.filter\|\.reduce" --include="*.{ts,js}" . | head -20

# Find nested loops (O(n²) potential)
grep -rn "for.*for\|\.forEach.*\.forEach\|\.map.*\.map" --include="*.{ts,js}" . | head -10

# Find sync operations that could be async
grep -rn "readFileSync\|writeFileSync\|execSync" --include="*.{ts,js}" . | head -10
```

#### 内存分析

```bash
# Large array operations
grep -rn "new Array\|Array\.from\|\.concat\|spread" --include="*.{ts,js}" . | head -10

# Potential memory leaks (event listeners, intervals)
grep -rn "addEventListener\|setInterval\|setTimeout" --include="*.{ts,js}" . | head -10
```

#### 数据库查询分析

```bash
# N+1 query patterns
grep -rn "await.*find\|await.*query" --include="*.{ts,js}" . | head -15

# Missing indexes hints
grep -rn "WHERE\|ORDER BY\|GROUP BY" --include="*.{ts,js,sql}" . | head -15
```

#### Bundle 分析

```bash
# Check bundle size (if applicable)
[ -f "package.json" ] && npm run build 2>/dev/null && ls -lh dist/*.js 2>/dev/null

# Large dependencies
[ -f "package.json" ] && cat package.json | jq '.dependencies | keys[]' | head -20
```

### 第三步：优先级排序

按以下维度对发现的问题排序：
1. **影响**：能提升多少性能？
2. **成本**：修复难度如何？
3. **风险**：可能破坏哪些内容？

## 输出格式

---

### ⚡ 性能分析

**目标**：[文件/模块/系统]
**分析时间**：[时间戳]

### 📊 当前指标（如可测量）

| 指标 | 当前 | 目标 | 差距 |
|--------|---------|--------|-----|
| 响应时间 | Xms | <Yms | 需降低 -Z% |
| 内存使用 | XMB | <YMB | 需降低 -Z% |
| Bundle 体积 | XKB | <YKB | 需降低 -Z% |

### 🔴 严重问题

#### 1. [问题标题] - [位置]

**问题**：[慢在哪里以及原因]

**当前**：
```typescript
// O(n²) - nested loops
users.forEach(user => {
  permissions.forEach(perm => {
    if (user.id === perm.userId) { ... }
  });
});
```

**优化后**：
```typescript
// O(n) - Map lookup
const permMap = new Map(permissions.map(p => [p.userId, p]));
users.forEach(user => {
  const perm = permMap.get(user.id);
  if (perm) { ... }
});
```

**影响**：1000 个用户时约快 10 倍
**成本**：低（5 分钟）
**风险**：低

### 🟠 高优先级

| 问题 | 位置 | 影响 | 成本 |
|-------|----------|--------|--------|
| [描述] | 文件:行 | [估算] | [时间] |

### 🟡 中优先级

| 问题 | 位置 | 影响 | 成本 |
|-------|----------|--------|--------|
| [描述] | 文件:行 | [估算] | [时间] |

### 💡 快速收益

1. [改动小但收益好的优化]
2. [另一个快速优化]
3. [唾手可得的改进]

### 📈 优化路线图

```
第 1 周：修复严重问题（第 1-3 项）
第 2 周：处理高优先级（第 4-6 项）
第 3 周：度量并验证改进效果
```

---

## 常见模式

### 数组操作

| 模式 | 问题 | 修复方案 |
|---------|-------|-----|
| `arr.filter().map()` | 两次遍历 | 改用单次 `reduce()` 或 `flatMap()` |
| 循环中使用 `arr.find()` | O(n²) | 先构建 Map/Set |
| `[...arr1, ...arr2]` | 内存分配 | 改用 `arr1.concat(arr2)` 或 push |

### 数据库

| 模式 | 问题 | 修复方案 |
|---------|-------|-----|
| 循环中使用 await | N+1 查询 | 改用 `IN` 批量查询 |
| `SELECT *` | 查询过多字段 | 只选取需要的列 |
| WHERE 条件缺少索引 | 全表扫描 | 添加复合索引 |

### React/前端

| 模式 | 问题 | 修复方案 |
|---------|-------|-----|
| JSX 中内联函数 | 触发重渲染 | 改用 `useCallback` |
| 渲染大列表 | DOM 频繁操作 | 虚拟化 |
| 图片未优化 | LCP 慢 | Next/Image、懒加载 |

### Node.js

| 模式 | 问题 | 修复方案 |
|---------|-------|-----|
| 同步文件操作 | 阻塞事件循环 | 改用异步方案 |
| `JSON.parse` 大文件 | 内存峰值 | 流式解析器 |
| 无连接池 | 连接开销大 | 使用 pg-pool 等连接池 |

## 用法

**分析特定文件：**
```
/optimize src/services/user.ts
```

**聚焦特定方向：**
```
/optimize --queries src/repositories/
/optimize --bundle
/optimize --memory src/workers/
```

**指定目标指标：**
```
/optimize --target=100ms src/api/search.ts
```

**快速扫描：**
```
/optimize --quick
```

## 注意事项

- 用测量代替假设：优化前先做性能分析
- 过早优化是万恶之源（Knuth）
- 聚焦热路径：优化高频执行的部分
- 权衡取舍：速度 vs 可读性 vs 可维护性

$ARGUMENTS
