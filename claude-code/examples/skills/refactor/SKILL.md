> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: refactor
description: 分析代码的 SOLID 原则违反情况并提出针对性改进建议
argument-hint: "<file_or_module> [--pattern <name>]"
effort: medium
disable-model-invocation: true
---

# SOLID 重构助手

分析代码的 SOLID 原则违反情况并提出针对性改进建议。

## 目的

基于以下维度识别重构机会：
- SOLID 原则违反
- 代码坏味道与反模式
- 复杂度度量
- 重复代码检测

## 使用说明

### 第一步：范围分析

根据用户输入确定重构范围：
- 单文件：深度分析
- 目录：跨文件模式检测
- 函数/类：针对性的提取建议

```bash
# 获取文件/目录统计
if [ -f "$TARGET" ]; then
  wc -l "$TARGET"
  echo "单文件分析"
elif [ -d "$TARGET" ]; then
  find "$TARGET" -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" \) | wc -l
  echo "目录分析"
fi
```

### 第二步：SOLID 违反检测

#### S - 单一职责原则

查找：
- 超过 300 行的文件
- 超过 50 行的函数
- 超过 10 个方法的类
- 混合关注点（数据 + UI + 业务逻辑）

```bash
# 查找大文件
find . -name "*.{ts,js,py}" -exec wc -l {} + 2>/dev/null | sort -rn | head -10

# 行数较多的函数（近似）
grep -rn "function\|def \|fn " --include="*.{ts,js,py,rs}" . | head -20
```

#### O - 开闭原则

查找：
- 基于类型的 switch/case 语句
- 重复的 if/else 类型判断
- 直接修改而非扩展

#### L - 里氏替换原则

查找：
- 抛出"未实现"的重写方法
- 方法调用前的类型检查
- 空的方法重写

#### I - 接口隔离原则

查找：
- 大型接口（超过 10 个方法）
- 类实现了接口中未使用的方法
- 臃肿的服务类

#### D - 依赖倒置原则

查找：
- 直接实例化依赖（`new Service()`）
- 硬编码的类引用
- 缺少依赖注入

### 第三步：代码坏味道

```bash
# 重复模式
grep -rn --include="*.{ts,js,py}" . 2>/dev/null | \
  awk -F: '{print $3}' | sort | uniq -c | sort -rn | head -10

# 过长的参数列表（超过 4 个参数）
grep -rn "function.*,.*,.*,.*," --include="*.{ts,js}" . 2>/dev/null | head -10

# 深层嵌套（4层以上）
grep -rn "^\s\{16,\}" --include="*.{ts,js,py}" . 2>/dev/null | head -10
```

### 第四步：复杂度评估

对发现的每个问题，评估：
- **影响范围**：受影响的代码量有多大？
- **风险**：可能导致什么问题？
- **工作量**：需要修改多少行，需要哪些测试？

## 输出格式

---

### 🔧 重构分析

**目标**：[文件/目录]
**分析行数**：[数量]

### 📊 SOLID 评分卡

| 原则 | 状态 | 发现问题 |
|-----------|--------|--------------|
| 单一职责 | 🟡 | 3 个大类 |
| 开闭原则 | 🟢 | 正常 |
| 里氏替换 | 🟢 | 正常 |
| 接口隔离 | 🔴 | 2 个臃肿接口 |
| 依赖倒置 | 🟡 | 5 处直接实例化 |

### 🎯 优先重构项

#### 1. [最高优先级] - 从 `UserService` 中提取类

**违反原则**：单一职责
**当前状态**：450 行代码同时处理认证 + 个人资料 + 通知
**建议方案**：
```
UserService.ts（450 行）
    ↓ 提取
AuthService.ts（约 150 行）
ProfileService.ts（约 150 行）
NotificationService.ts（约 100 行）
```
**风险**：中等（需更新 import）
**所需测试**：更新测试中的依赖注入

#### 2. [次优先级] - 用多态替换 switch

**位置**：`src/handlers/payment.ts:45`
**当前代码**：
```typescript
switch (paymentType) {
  case 'card': // 50 行
  case 'bank': // 50 行
  case 'crypto': // 50 行
}
```
**建议方案**：使用 `PaymentProcessor` 接口实现策略模式
**风险**：低（变更隔离）

### 📝 代码坏味道

| 坏味道 | 位置 | 严重程度 |
|-------|----------|----------|
| 过长方法 | `api.ts:calculateTotal`（120 行） | 🟠 高 |
| 重复代码 | `utils/*.ts`（3 个相似代码块） | 🟡 中 |
| 深层嵌套 | `parser.ts:parse`（6 层） | 🟡 中 |

### 🚀 快速改进（低风险，高价值）

1. 将 `validateEmail()` 提取到共享工具函数（已在 4 处使用）
2. 用命名常量替换魔法数字
3. 在 `processOrder()` 中添加提前返回以减少嵌套

### ⚠️ 技术债务备注

- [需在后续迭代中跟踪的事项]

---

## 重构安全检查清单

应用建议前：

- [ ] 受影响代码已有测试覆盖
- [ ] 创建功能分支
- [ ] 提交当前状态
- [ ] 每次只应用一项重构
- [ ] 每次变更后运行测试
- [ ] 提交前审查差异

## 用法

**分析指定文件：**
```
/refactor src/services/user.ts
```

**分析目录：**
```
/refactor src/api/
```

**聚焦特定原则：**
```
/refactor --focus=srp src/services/
```

**设定复杂度阈值：**
```
/refactor --threshold=high
```

## 参考资料

- Martin Fowler 的《重构》目录
- Robert C. Martin 的《代码整洁之道》
- Robert C. Martin 的 SOLID 原则

$ARGUMENTS
