> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: generate-tests
description: 为指定代码生成全面的测试用例
argument-hint: "<file_or_module> [--framework jest|vitest|pytest]"
effort: medium
disable-model-invocation: true
---

# 生成测试

为指定代码生成全面的测试用例。

## 操作说明

1. 读取目标文件
2. 识别可测试单元（函数、类、方法）
3. 按照项目规范生成测试
4. 确保边界情况有高覆盖率

## 测试生成流程

### 1. 分析目标
- 识别公共接口
- 理解依赖关系
- 标注边界情况与边界值

### 2. 检测测试框架
检查以下文件：
- `jest.config.js` → Jest
- `vitest.config.ts` → Vitest
- `pytest.ini` → pytest
- `package.json` 中含 `mocha` → Mocha

### 3. 生成测试
遵循检测到的框架规范。

## 测试分类

### 正常路径
使用有效输入的正常预期行为。

### 边界情况
- 空输入
- Null/undefined 值
- 边界值（0、-1、MAX_INT）
- 单项 vs 多项

### 错误情况
- 无效输入类型
- 缺少必填参数
- 网络/IO 故障
- 超时场景

### 集成点
- 数据库交互
- 外部 API 调用
- 文件系统操作

## 输出格式

```typescript
describe('[ComponentName]', () => {
  describe('[methodName]', () => {
    // Happy path
    it('should [expected behavior] when [condition]', () => {
      // Arrange
      // Act
      // Assert
    });

    // Edge cases
    it('should handle empty input', () => {});
    it('should handle null values', () => {});

    // Error cases
    it('should throw when [invalid condition]', () => {});
  });
});
```

## 编写规范

- 每个测试一个断言（在实际可行时）
- 描述性测试名称
- AAA 模式（Arrange-Act-Assert）
- 测试之间不相互依赖
- Mock 外部依赖

## 使用方式

```
/generate-tests src/utils/calculator.ts
/generate-tests src/services/
```

$ARGUMENTS
