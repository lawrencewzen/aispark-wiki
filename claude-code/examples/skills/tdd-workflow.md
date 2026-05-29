> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: tdd-workflow
description: 测试驱动开发工作流与最佳实践
effort: low
---

# TDD 工作流 Skill

## TDD 循环

```
RED → GREEN → REFACTOR
 ↑__________________|
```

### 1. RED：编写失败的测试
- 编写能够失败的最小测试
- 测试应因正确原因而失败
- 确保测试确实能运行

### 2. GREEN：让测试通过
- 编写使测试通过的最简实现
- 暂不优化
- 代码暂时难看也没关系

### 3. REFACTOR：清理代码
- 改善代码结构
- 消除重复
- 保持测试持续通过

## TDD 最佳实践

### 测试命名规范
```
should_[预期行为]_when_[条件]
```

示例：
- `should_return_empty_array_when_no_items`
- `should_throw_error_when_invalid_input`
- `should_calculate_total_when_items_present`

### 测试结构（AAA）
```typescript
it('should calculate discount when coupon applied', () => {
  // Arrange - 准备测试数据
  const cart = new Cart();
  cart.addItem({ price: 100 });
  const coupon = new Coupon('10OFF', 10);

  // Act - 执行目标行为
  cart.applyCoupon(coupon);

  // Assert - 验证结果
  expect(cart.total).toBe(90);
});
```

### 测试隔离
- 每个测试应相互独立
- 测试之间不共享状态
- 使用 `beforeEach` 进行公共初始化
- 在 `afterEach` 中进行清理

## TDD 工作流示例

### 功能：向购物车添加商品

**步骤 1：RED**
```typescript
describe('Cart', () => {
  it('should add item to cart', () => {
    const cart = new Cart();
    cart.addItem({ id: 1, name: 'Book', price: 29.99 });
    expect(cart.items).toHaveLength(1);
  });
});
```
运行测试 → 失败（Cart 不存在）

**步骤 2：GREEN**
```typescript
class Cart {
  items = [];

  addItem(item) {
    this.items.push(item);
  }
}
```
运行测试 → 通过

**步骤 3：REFACTOR**
```typescript
class Cart {
  private _items: CartItem[] = [];

  get items(): ReadonlyArray<CartItem> {
    return this._items;
  }

  addItem(item: CartItem): void {
    this._items.push(item);
  }
}
```
运行测试 → 仍然通过

### 下一轮迭代：计算总价
对每个新行为重复上述循环。

## 何时使用 TDD

### 适合 TDD
- 业务逻辑
- 复杂算法
- API 端点
- 状态管理
- 工具函数

### 不太适合
- UI 布局（视觉测试更合适）
- 数据库迁移
- 外部集成（使用集成测试）
- 探索性/原型代码

## TDD 常见误区

1. **测试写得太多** — 从最小的失败测试开始
2. **代码写得太多** — 只写足以让测试通过的代码
3. **跳过重构步骤** — 技术债会积累
4. **测试实现细节** — 测试行为，而非内部实现
5. **忽略失败的测试** — 必须修复或删除，绝不跳过

## 测试替身

| 类型 | 用途 | 示例 |
|------|---------|---------|
| Stub | 返回固定数据 | `jest.fn().mockReturnValue(42)` |
| Mock | 验证交互行为 | `expect(mock).toHaveBeenCalled()` |
| Spy | 追踪调用记录 | `jest.spyOn(obj, 'method')` |
| Fake | 简化实现 | 内存数据库 |
