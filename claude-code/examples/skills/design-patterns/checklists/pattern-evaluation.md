> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "设计模式质量评估检查清单"
description: "用于评估设计模式实现质量的系统化评分标准"
tags: [cheatsheet, design-patterns, code-review]
---

# 设计模式质量评估检查清单

用于评估设计模式实现质量的系统化标准。

## 评估标准

每项标准评分范围为 **0-10**，其中：
- **9-10**：典范级，可作为参考实现的质量
- **7-8**：良好，有小幅改进空间
- **5-6**：可接受，存在需要处理的明显问题
- **3-4**：有问题，需要大规模重构
- **0-2**：错误或存在严重缺陷

**综合得分** = 所有标准分数的平均值

---

## 1. 正确性（0-10）

**问题**：实现是否正确遵循了规范的模式结构？

### 评分指南

| 分数 | 描述 |
|-------|-------------|
| 9-10  | 完全遵循模式结构，所有角色均存在且正确实现 |
| 7-8   | 存在轻微偏差，但不影响模式意图 |
| 5-6   | 存在一些结构问题，但模式可识别且功能正常 |
| 3-4   | 存在重大结构问题，模式仅部分实现 |
| 0-2   | 实现错误，与模式完全不符 |

### 检查清单

**单例模式（Singleton）**：
- [ ] 私有构造函数
- [ ] 静态 getInstance() 方法
- [ ] 私有静态实例字段
- [ ] 线程安全（如适用）
- [ ] 每次返回相同实例

**观察者模式（Observer）**：
- [ ] Subject 接口包含 attach/detach/notify 方法
- [ ] Observer 接口包含 update 方法
- [ ] Subject 维护观察者列表
- [ ] notify 调用所有观察者的 update
- [ ] 观察者可以动态添加和移除

**策略模式（Strategy）**：
- [ ] Strategy 接口定义算法
- [ ] Context 持有策略引用
- [ ] Context 委托给策略执行
- [ ] 各策略可互换
- [ ] 客户端可以在运行时设置策略

**工厂方法模式（Factory Method）**：
- [ ] 工厂方法返回接口或抽象类
- [ ] 子类重写工厂方法
- [ ] 客户端代码依赖接口而非具体类
- [ ] 创建逻辑被封装

**装饰器模式（Decorator）**：
- [ ] 装饰器实现与被包装对象相同的接口
- [ ] 装饰器持有被包装对象的引用
- [ ] 装饰器委托给被包装对象
- [ ] 可以叠加多个装饰器
- [ ] 保持接口契约

### 常见错误（扣分项）

- **-2**：缺少关键组件（例如，单例模式缺少私有构造函数）
- **-3**：委托不正确（例如，装饰器未调用被包装对象）
- **-4**：破坏模式不变量（例如，单例模式返回不同实例）

---

## 2. 可测试性（0-10）

**问题**：为该实现编写单元测试的难易程度如何？

### 评分指南

| 分数 | 描述 |
|-------|-------------|
| 9-10  | 易于模拟，依赖可注入，无全局状态 |
| 7-8   | 仅需少量准备即可测试，存在一定耦合 |
| 5-6   | 需要大量测试准备，耦合程度中等 |
| 3-4   | 难以测试，紧耦合，存在全局状态 |
| 0-2   | 几乎不可测试，静态依赖，无注入点 |

### 检查清单

- [ ] 依赖通过注入提供（而非内部创建）
- [ ] 使用接口（可被模拟）
- [ ] 无硬编码依赖
- [ ] 无全局状态访问（或最小化）
- [ ] 测试可以相互隔离（测试之间互不影响）
- [ ] 无无法被模拟的静态方法
- [ ] 副作用最小化或可控

### 危险信号（扣分项）

- **-2**：使用 `getInstance()` 而非依赖注入
- **-2**：硬编码具体类实例化
- **-3**：访问全局状态（业务逻辑中使用 window、global、process.env）
- **-3**：带副作用的静态方法
- **-4**：无法注入测试替身

### 示例

**可测试性差（得分：2/10）**：
```typescript
class PaymentService {
  processPayment(amount: number) {
    // 难以测试：内部创建依赖
    const gateway = PaymentGateway.getInstance();
    // 难以测试：访问全局配置
    const apiKey = process.env.PAYMENT_API_KEY;
    return gateway.charge(amount, apiKey);
  }
}
```

**可测试性好（得分：9/10）**：
```typescript
class PaymentService {
  constructor(
    private gateway: IPaymentGateway,
    private config: Config
  ) {}

  processPayment(amount: number) {
    const apiKey = this.config.getPaymentApiKey();
    return this.gateway.charge(amount, apiKey);
  }
}

// 使用模拟对象轻松测试
const mockGateway = { charge: jest.fn() };
const mockConfig = { getPaymentApiKey: () => 'test-key' };
const service = new PaymentService(mockGateway, mockConfig);
```

---

## 3. 单一职责原则（0-10）

**问题**：组件是否拥有一个明确定义的职责？

### 评分指南

| 分数 | 描述 |
|-------|-------------|
| 9-10  | 职责单一且聚焦，类只有一个变更原因 |
| 7-8   | 职责基本聚焦，存在少量次要关注点 |
| 5-6   | 存在多个相关职责 |
| 3-4   | 存在多个不相关职责 |
| 0-2   | 上帝类，承担过多职责 |

### 检查清单

- [ ] 类或模块有一个明确的目的
- [ ] 所有方法均与主要职责相关
- [ ] 更改某个需求不会导致必须修改此类
- [ ] 类名清晰反映其职责
- [ ] 类名或描述中不含"and"（例如，"UserManagerAndLogger"是不好的做法）

### 危险信号（扣分项）

- **-2**：类处理 2 个不同的关注点
- **-3**：类处理 3 个及以上关注点
- **-4**：上帝类（方法超过 20 个，代码超过 300 行）
- **-1**：存在与主要职责无关的方法

### 示例

**单一职责原则较差（得分：3/10）**：
```typescript
class UserService {
  createUser(data: UserData) { /* ... */ }
  validateEmail(email: string) { /* ... */ }
  sendWelcomeEmail(user: User) { /* ... */ }
  logUserActivity(activity: string) { /* ... */ }
  generateReport(userId: string) { /* ... */ }
  // 职责过多：创建、验证、发邮件、日志记录、报告生成
}
```

**单一职责原则良好（得分：9/10）**：
```typescript
class UserService {
  constructor(
    private validator: UserValidator,
    private emailService: EmailService,
    private logger: Logger
  ) {}

  createUser(data: UserData): User {
    // 仅专注于用户创建的编排
    this.validator.validate(data);
    const user = new User(data);
    this.emailService.sendWelcomeEmail(user);
    this.logger.logActivity('user_created', user.id);
    return user;
  }
}
```

---

## 4. 开闭原则（0-10）

**问题**：组件是否可以在不修改源代码的情况下进行扩展？

### 评分指南

| 分数 | 描述 |
|-------|-------------|
| 9-10  | 完全可通过继承或组合进行扩展，无需修改 |
| 7-8   | 大体可扩展，可能需要少量修改 |
| 5-6   | 存在一定扩展点但受限 |
| 3-4   | 难以扩展，需要在多处进行修改 |
| 0-2   | 对扩展封闭，必须修改源代码 |

### 检查清单

- [ ] 使用接口或抽象类
- [ ] 新行为可通过新增类而非修改现有类来添加
- [ ] 使用配置或策略模式来支持行为变化
- [ ] 无基于类型的 switch 语句（添加新类型需要修改代码）
- [ ] 依赖倒置（依赖于抽象）

### 危险信号（扣分项）

- **-2**：基于类型的 switch（添加新类型需要修改）
- **-3**：无接口（处处依赖具体实现）
- **-3**：硬编码行为（无扩展点）
- **-4**：添加功能的唯一方式是修改现有方法

### 示例

**对扩展封闭（得分：2/10）**：
```typescript
class PaymentProcessor {
  process(type: string, amount: number) {
    switch (type) {
      case 'credit': return this.processCreditCard(amount);
      case 'paypal': return this.processPaypal(amount);
      // 添加加密货币支付需要修改此类
    }
  }
}
```

**对扩展开放（得分：9/10）**：
```typescript
interface PaymentStrategy {
  process(amount: number): Promise<Receipt>;
}

class PaymentProcessor {
  constructor(private strategies: Map<string, PaymentStrategy>) {}

  process(type: string, amount: number) {
    const strategy = this.strategies.get(type);
    if (!strategy) throw new Error(`Unknown payment type: ${type}`);
    return strategy.process(amount);
  }
}

// 无需修改 PaymentProcessor 即可添加新支付方式
class CryptoPaymentStrategy implements PaymentStrategy {
  process(amount: number) { /* ... */ }
}
```

---

## 5. 文档（0-10）

**问题**：实现是否有完善的文档，清晰说明意图和用法？

### 评分指南

| 分数 | 描述 |
|-------|-------------|
| 9-10  | 文档完整：意图、用法、示例、边界情况均有说明 |
| 7-8   | 文档良好，涵盖主要使用场景 |
| 5-6   | 基础文档，有但最少 |
| 3-4   | 文档稀少，意图不明 |
| 0-2   | 无文档或存在误导性文档 |

### 检查清单

- [ ] 类或接口有 JSDoc/TSDoc 注释说明其用途
- [ ] 模式意图已记录（"这是一个单例，因为……"）
- [ ] 公共方法有文档说明
- [ ] 复杂逻辑有行内注释
- [ ] 提供了使用示例（在 README 或注释中）
- [ ] 不变量和约束已记录
- [ ] 命名具有自文档性（清晰、描述性的名称）

### 危险信号（扣分项）

- **-2**：无类级文档
- **-2**：公共 API 方法未记录
- **-3**：晦涩命名（x、foo、temp、data）
- **-1**：复杂逻辑缺乏说明

### 示例

**文档质量差（得分：2/10）**：
```typescript
class S {
  private static i: S;
  private constructor() {}
  static get() {
    if (!S.i) S.i = new S();
    return S.i;
  }
  do(x: any) { /* ... */ }
}
```

**文档质量好（得分：9/10）**：
```typescript
/**
 * 以单例模式实现的配置服务，确保所有组件
 * 共享相同的配置状态。
 *
 * 使用 `ConfigService.getInstance()` 访问单例实例。
 *
 * @example
 * const config = ConfigService.getInstance();
 * const apiUrl = config.get('API_URL');
 */
class ConfigService {
  private static instance: ConfigService;

  /**
   * 私有构造函数，防止直接实例化。
   * 请使用 `getInstance()` 代替。
   */
  private constructor() {
    // 从环境变量加载配置
  }

  /**
   * 返回 ConfigService 的单例实例。
   * 首次调用时创建实例（懒初始化）。
   *
   * @returns ConfigService 单例实例
   */
  public static getInstance(): ConfigService {
    if (!ConfigService.instance) {
      ConfigService.instance = new ConfigService();
    }
    return ConfigService.instance;
  }

  /**
   * 通过键名获取配置值。
   *
   * @param key - 配置键名
   * @returns 配置值，若不存在则返回 undefined
   */
  public get(key: string): string | undefined {
    return process.env[key];
  }
}
```

---

## 模式专项评估

### 单例模式专项

**附加检查清单**：
- [ ] 懒初始化（如适用）
- [ ] 考虑线程安全（在 JavaScript 中不太关键）
- [ ] 子类化已被阻止或受控
- [ ] 无公共构造函数
- [ ] 提供测试用的重置机制（或提及 DI 替代方案）

**扣分项**：
- **-3**：公共构造函数（破坏模式目的）
- **-2**：多个 getInstance() 方法返回不同实例
- **-2**：未考虑测试隔离

### 观察者模式专项

**附加检查清单**：
- [ ] 观察者可以取消订阅
- [ ] 无内存泄漏（观察者被正确移除）
- [ ] 通知顺序是确定的（如有必要）
- [ ] 观察者不依赖通知顺序
- [ ] Subject 不知道具体的观察者类型

**扣分项**：
- **-3**：无取消订阅机制（存在内存泄漏风险）
- **-2**：Subject 依赖具体观察者类型
- **-2**：通知顺序有影响但未保证

### 策略模式专项

**附加检查清单**：
- [ ] 各策略实现共同接口
- [ ] Context 不依赖具体策略
- [ ] 各策略可互换
- [ ] 策略可在运行时设置
- [ ] 各策略不共享状态（除非有明确设计）

**扣分项**：
- **-3**：Context 依赖具体策略
- **-2**：各策略无法真正互换
- **-2**：无法在运行时更改策略

---

## 综合评分公式

```
综合得分 = (
  正确性 × 0.30 +
  可测试性 × 0.25 +
  单一职责 × 0.20 +
  开闭原则 × 0.15 +
  文档 × 0.10
) / 5
```

**加权说明**：正确性最重要，其次是可测试性。

---

## 解读指南

| 综合得分 | 解读 | 建议操作 |
|--------------|----------------|--------|
| 9.0 - 10.0   | 优秀 | 参考级质量，几乎无需改动 |
| 7.0 - 8.9    | 良好 | 小幅改进，可生产使用 |
| 5.0 - 6.9    | 可接受 | 存在明显问题，建议重构 |
| 3.0 - 4.9    | 较差 | 存在重大问题，必须重构 |
| 0.0 - 2.9    | 严重 | 存在根本性缺陷，需要重新设计 |

---

## 评估报告示例

### 模式：单例模式（Singleton）
**文件**：`src/services/config-singleton.ts`
**行数**：5-34

#### 得分

| 标准 | 得分 | 理由 |
|-----------|-------|-----------|
| 正确性 | 8/10 | 正确实现单例，轻微问题：未考虑线程安全（在 JavaScript 中不关键） |
| 可测试性 | 4/10 | 难以模拟，无重置机制，全局状态导致测试相互依赖 |
| 单一职责 | 9/10 | 仅专注于配置管理 |
| 开闭原则 | 7/10 | 可添加新配置键，但配置来源被硬编码 |
| 文档 | 6/10 | 有 JSDoc 但缺少选择单例模式的理由说明 |

**综合得分**：**6.8/10**（可接受）

#### 已识别问题

1. **高优先级**：添加 `resetInstance()` 方法以实现测试隔离
   - 当前状况：测试必须按特定顺序运行
   - 修复方案：添加受环境检查保护的 `public static resetInstance()`

2. **中优先级**：记录单例选择的理由
   - 当前状况：不清楚为何需要全局状态
   - 修复方案：添加 JSDoc 说明选择原因（例如，"单例确保所有服务使用一致的配置"）

3. **低优先级**：考虑依赖注入替代方案
   - 当前状况：难以测试，紧耦合
   - 建议：评估使用 DI 容器或 React Context

#### 建议

```typescript
// 添加测试友好的重置方法
public static resetInstance(): void {
  if (process.env.NODE_ENV === 'test') {
    ConfigService.instance = null!;
  }
}

// 更佳方案：重构为 DI
class ConfigService {
  constructor(private envVars: EnvVars) {}
}

// 在 DI 容器或 provider 中注入
const config = new ConfigService(process.env);
```

---

## 在技能中的使用方式

当设计模式技能以**评估模式**运行时，它会：

1. 识别实现的是哪种模式
2. 应用相关检查清单项
3. 对每个标准评分（0-10）
4. 计算加权综合得分
5. 生成详细报告，包含：
   - 得分表格
   - 发现的问题（按优先级排序）
   - 附代码示例的具体建议
   - 适用时提供技术栈原生替代方案

---

## 参考资料

- *Clean Code*，作者 Robert C. Martin（SOLID 原则）
- *Refactoring: Improving the Design of Existing Code*，作者 Martin Fowler
- *Design Patterns: Elements of Reusable Object-Oriented Software*（Gang of Four）
