> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: design-patterns
description: "检测、建议和评估 TypeScript/JavaScript 代码库中的 GoF 设计模式。适用于重构代码、应用单例/工厂/观察者/策略模式、审查模式质量，或为 React、Angular、NestJS、Vue 寻找技术栈原生替代方案。"
allowed-tools: Read, Grep, Glob, mcp__grepai__grepai_search
context: fork
agent: specialist
effort: high
---

# 设计模式分析技能

**用途**：检测、建议和评估 TypeScript/JavaScript 代码库中的四人组（GoF）设计模式，并提供技术栈感知的适配方案。

## 核心能力

1. **技术栈检测**：识别主要框架/库（React、Angular、NestJS、Vue、Express、RxJS、Redux、ORM）
2. **模式检测**：发现 23 种 GoF 模式的现有实现
3. **智能建议**：推荐模式以修复代码异味，优先使用技术栈原生惯用法
4. **质量评估**：依照最佳实践评定模式实现质量

## 运行模式

### 模式一：检测

**触发条件**：用户请求模式检测或分析
**输出**：包含置信度评分和技术栈上下文的 JSON 报告

**工作流程**：
```
1. 技术栈检测（package.json、tsconfig.json、框架文件）
2. 模式搜索（Glob 查找候选文件 → Grep 匹配特征 → Read 验证）
3. 分类（技术栈原生 vs 自定义实现）
4. 置信度评分（0.0-1.0，基于检测规则）
5. 生成 JSON 报告
```

**调用示例**：
```
/design-patterns detect src/
/design-patterns analyze --format=json
```

### 模式二：建议

**触发条件**：用户请求模式建议或重构建议
**输出**：带优先级建议和技术栈适配示例的 Markdown 报告

**工作流程**：
```
1. 代码异味检测（switch 语句、过长参数列表、全局状态等）
2. 模式匹配（将异味映射到适用模式）
3. 技术栈适配（优先使用框架原生模式而非自定义实现）
4. 优先级排序（影响力 × 可行性）
5. 生成带代码示例的 Markdown 报告
```

**调用示例**：
```
/design-patterns suggest src/payment/
/design-patterns refactor --focus=creational
```

### 模式三：评估

**触发条件**：用户请求模式质量评估
**输出**：包含各评估标准评分的 JSON 报告

**工作流程**：
```
1. 模式识别（确定实现了哪种模式）
2. 标准评估（正确性、可测试性、SOLID 合规性、文档）
3. 问题检测（常见错误、反模式）
4. 评分（每项标准 0-10 分）
5. 生成带建议的 JSON 报告
```

**调用示例**：
```
/design-patterns evaluate src/services/singleton.ts
/design-patterns quality --pattern=observer
```

## 方法论

### 第一阶段：技术栈检测

**数据来源**（按优先级排序）：
1. `package.json` → 检查 dependencies 和 devDependencies
2. 框架专属文件 → `angular.json`、`next.config.*`、`nest-cli.json`、`vite.config.*`
3. `tsconfig.json` → 检查 compilerOptions、paths、lib
4. 文件扩展名 → `*.jsx`、`*.tsx`、`*.vue` 的存在

**检测规则**（来自 `signatures/stack-patterns.yaml`）：
- React：deps 中有 `react` + `*.jsx/*.tsx` 文件
- Angular：有 `@angular/core` + `angular.json`
- NestJS：有 `@nestjs/core` + `nest-cli.json`
- Vue：`vue` v3+ + `*.vue` 文件
- Express：deps 中有 `express` + `app.use` 模式
- RxJS：deps 中有 `rxjs` + Observable 用法
- Redux/Zustand：deps 中有 `redux`/`zustand` + store 模式
- Prisma/TypeORM：deps 中有 `prisma`/`typeorm` + schema 文件

**输出**：
```json
{
  "stack_detected": {
    "primary": "react",
    "version": "18.2.0",
    "secondary": ["typescript", "zustand", "prisma"],
    "detection_sources": ["package.json", "tsconfig.json", "37 *.tsx files"],
    "confidence": 0.95
  }
}
```

### 第二阶段：模式检测

**搜索策略**：
1. **Glob 阶段**：按命名约定查找候选文件
   - `*Singleton*.ts`、`*Factory*.ts`、`*Strategy*.ts`、`*Observer*.ts` 等
   - `*Manager*.ts`、`*Builder*.ts`、`*Adapter*.ts`、`*Proxy*.ts` 等

2. **Grep 阶段**：搜索模式特征（来自 `signatures/detection-rules.yaml`）
   - 主要信号：`private constructor`、`static getInstance()`、`subscribe()`、`createXxx()` 等
   - 次要信号：接口命名、委托模式、方法签名

3. **Read 阶段**：验证模式结构
   - 解析类/接口定义
   - 验证关系（继承、组合、委托）
   - 检查是完整实现还是部分使用

**置信度评分**：
- 0.9-1.0：所有主次信号均存在，结构完全匹配
- 0.7-0.89：所有主要信号 + 部分次要信号，轻微偏差
- 0.5-0.69：主要信号存在，缺少次要验证
- 0.3-0.49：命名约定匹配，结构证据较弱
- 0.0-0.29：证据不足，可能为误判

**分类**：
- `native`：使用技术栈原生特性实现的模式（React Context、Angular Services、NestJS Guards 等）
- `custom`：手动 TypeScript 实现
- `library`：第三方库提供的模式（RxJS Subject、Redux Store 等）

### 第三阶段：代码异味检测

**目标异味**（来自 `signatures/code-smells.yaml`）：
1. **基于类型的 switch** → 策略/工厂模式
2. **过长参数列表（>4 个）** → 建造者模式
3. **全局状态访问** → 单例（或优先使用依赖注入）
4. **重复的状态条件判断** → 状态模式
5. **分散的通知逻辑** → 观察者模式
6. **复杂的对象创建** → 工厂/抽象工厂
7. **与具体类的紧耦合** → 适配器/桥接
8. **重复的接口转换** → 适配器模式
9. **深度嵌套的功能扩展** → 装饰器模式
10. **职责过多的大类** → 外观模式

**检测启发式规则**：
- Grep 搜索 `switch (.*type)`、`switch (.*kind)`、`switch (.*mode)`
- 统计函数参数数量：`function \w+\([^)]{60,}\)`（>4 个参数的近似匹配）
- 搜索全局访问：`window\.`、`global\.`、`process\.env\.\w+`（不含配置文件中的用法）
- 查找状态条件：`if.*state.*===.*&&.*if.*state.*===`
- 查找通知模式：`forEach.*notify`、`map.*\.emit\(`

### 第四阶段：技术栈感知建议

**适配逻辑**（来自 `signatures/stack-patterns.yaml`）：

```
IF 检测到自定义模式 AND 技术栈有原生等效方案:
  建议："使用技术栈原生模式替代"
  提供：当前方案 vs 推荐方案的对比

ELSE IF 检测到代码异味 AND 缺少模式:
  IF 技术栈提供该模式:
    建议：带示例的技术栈原生实现
  ELSE:
    建议：TypeScript 自定义实现及最佳实践

ELSE IF 模式实现有误:
  提供：修复反模式的重构步骤
```

**适配示例**：

| 模式 | 技术栈 | 原生替代方案 | 建议 |
|---------|-------|-------------------|----------------|
| 单例 | React | Context API + Provider | 使用 `createContext()` 替代 `getInstance()` |
| 观察者 | Angular | RxJS Subject/BehaviorSubject | 使用内置 Observable，不要自定义实现 |
| 装饰器 | NestJS | @Injectable() 装饰器 + 拦截器 | 使用框架拦截器 |
| 策略 | Vue 3 | Composition API 组合式函数 | 使用 `ref()` + 组合式函数替代类 |
| 职责链 | Express | 中间件（`app.use()`） | 使用 Express 中间件链 |
| 命令 | Redux | Action creators + reducers | 使用 Redux actions，不要自定义命令对象 |

### 第五阶段：质量评估

**标准**（来自 `checklists/pattern-evaluation.md`）：
1. **正确性（0-10）**：是否符合规范的模式结构？
2. **可测试性（0-10）**：依赖项是否易于 mock/stub？
3. **单一职责（0-10）**：是否只做一件事？
4. **开闭原则（0-10）**：是否无需修改即可扩展？
5. **文档（0-10）**：意图是否清晰，命名是否有描述性？

**评分指南**：
- 9-10：优秀，参考级别的实现
- 7-8：良好，有小幅改进空间
- 5-6：可接受，存在值得关注的问题
- 3-4：有问题，需要较大范围重构
- 0-2：实现错误或严重缺陷

**问题检测**：
- 硬编码依赖（在 getInstance 内部使用 new 的单例）
- 上帝类（职责过多）
- 抽象泄漏（暴露内部结构）
- 缺少错误处理
- 命名不当（Strategy1、Strategy2 而非描述性名称）

## 输出格式

### 检测模式（JSON）

```json
{
  "metadata": {
    "scan_date": "2026-01-21T10:30:00Z",
    "scope": "src/",
    "files_scanned": 147,
    "execution_time_ms": 2341
  },
  "stack_detected": {
    "primary": "react",
    "version": "18.2.0",
    "secondary": ["typescript", "zustand", "prisma"],
    "detection_sources": ["package.json", "tsconfig.json", "37 *.tsx files"],
    "confidence": 0.95
  },
  "patterns_found": {
    "singleton": [
      {
        "file": "src/lib/api-client.ts",
        "lines": "5-28",
        "confidence": 0.85,
        "type": "custom",
        "signals": ["private constructor", "static getInstance", "private static instance"],
        "note": "Consider using React Context instead for better testability"
      }
    ],
    "observer": [
      {
        "file": "src/hooks/useAuth.ts",
        "lines": "12-45",
        "confidence": 0.92,
        "type": "native",
        "implementation": "React useState + useEffect",
        "note": "Correctly using React's built-in observer pattern"
      }
    ],
    "factory": [
      {
        "file": "src/services/notification-factory.ts",
        "lines": "8-67",
        "confidence": 0.78,
        "type": "custom",
        "signals": ["createNotification method", "type discrimination", "returns interface"]
      }
    ]
  },
  "summary": {
    "total_patterns": 7,
    "native_to_stack": 4,
    "custom_implementations": 3,
    "by_category": {
      "creational": 2,
      "structural": 3,
      "behavioral": 2
    },
    "by_confidence": {
      "high": 5,
      "medium": 2,
      "low": 0
    }
  },
  "recommendations": [
    "Consider replacing custom Singleton (api-client.ts) with React Context for better DI",
    "Review Factory pattern (notification-factory.ts) - could be simplified with strategy pattern"
  ]
}
```

### 建议模式（Markdown）

```markdown
# 设计模式建议

**范围**：`src/payment/`
**技术栈**：React 18 + TypeScript + Stripe
**日期**：2026-01-21

---

## 高优先级

### 1. 策略模式 → `src/payment/processor.ts:45-89`

**代码异味**：基于支付类型的 switch 语句（4 个分支，78 行）

**当前实现**（第 52-87 行）：
```typescript
switch (paymentType) {
  case 'credit':
    // 20 行信用卡逻辑
    break;
  case 'paypal':
    // 15 行 PayPal 逻辑
    break;
  case 'crypto':
    // 18 行加密货币逻辑
    break;
  case 'bank':
    // 12 行银行转账逻辑
    break;
}
```

**推荐方案（React 适配的策略模式）**：
```typescript
// 定义策略接口
interface PaymentStrategy {
  process: (amount: number) => Promise<PaymentResult>;
}

// 自定义 hook 作为策略
const useCreditPayment = (): PaymentStrategy => ({
  process: async (amount) => { /* 信用卡逻辑 */ }
});

const usePaypalPayment = (): PaymentStrategy => ({
  process: async (amount) => { /* PayPal 逻辑 */ }
});

// 策略选择 hook
const usePaymentStrategy = (type: PaymentType): PaymentStrategy => {
  const strategies = {
    credit: useCreditPayment(),
    paypal: usePaypalPayment(),
    crypto: useCryptoPayment(),
    bank: useBankPayment(),
  };
  return strategies[type];
};

// 在组件中使用
const PaymentForm = ({ type }: Props) => {
  const strategy = usePaymentStrategy(type);
  const handlePay = () => strategy.process(amount);
  // ...
};
```

**收益**：
- **复杂度**：圈复杂度从 12 降低到 2
- **可扩展性**：新增支付方式 = 新增 hook，无需修改现有代码
- **可测试性**：每个策略 hook 可独立测试
- **工作量**：约 2 小时（将逻辑提取到 hook 并补充测试）

---

## 中优先级

### 2. 观察者模式 → `src/cart/CartManager.ts:23-156`

**代码异味**：手动通知逻辑分散在 8 个方法中

**当前方案**：手动循环调用更新函数
**推荐方案**：使用 Zustand store（已在依赖中）

```typescript
// 替代自定义观察者：
import create from 'zustand';

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  // Zustand 自动通知订阅者
}

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({ items: state.items.filter(i => i.id !== id) })),
}));

// 组件自动订阅：
const CartDisplay = () => {
  const items = useCartStore((state) => state.items);
  // 购物车变化时自动重新渲染
};
```

**收益**：
- **代码行数**：从 156 行减少到约 25 行
- **技术栈原生**：使用现有 Zustand 依赖
- **可测试性**：Zustand store 易于测试
- **工作量**：约 1.5 小时

---

## 汇总

- **建议总数**：4
- **高优先级**：2（策略、观察者）
- **中优先级**：2（建造者、外观）
- **预计总工作量**：约 6 小时
- **主要收益**：降低复杂度、提升可测试性、使用技术栈原生惯用法
```

### 评估模式（JSON）

```json
{
  "file": "src/services/config-singleton.ts",
  "pattern": "singleton",
  "lines": "5-34",
  "scores": {
    "correctness": 8,
    "testability": 4,
    "single_responsibility": 9,
    "open_closed": 7,
    "documentation": 6,
    "overall": 6.8
  },
  "details": {
    "correctness": {
      "score": 8,
      "rationale": "Implements singleton structure correctly with private constructor and static getInstance",
      "issues": ["Missing thread-safety consideration (not critical in JS single-threaded context)"]
    },
    "testability": {
      "score": 4,
      "rationale": "Hard to mock or reset instance in tests",
      "issues": [
        "No reset method for test isolation",
        "Static instance makes dependency injection impossible",
        "Tests must run in specific order or share state"
      ],
      "suggestions": [
        "Add resetInstance() method for tests (with appropriate guards)",
        "Consider using dependency injection instead"
      ]
    },
    "single_responsibility": {
      "score": 9,
      "rationale": "Focuses solely on configuration management",
      "issues": []
    },
    "open_closed": {
      "score": 7,
      "rationale": "Configuration can be extended but requires modification for new sources",
      "suggestions": ["Consider strategy pattern for configuration sources"]
    },
    "documentation": {
      "score": 6,
      "rationale": "Has JSDoc but missing rationale for singleton choice",
      "suggestions": ["Document why singleton is chosen over DI", "Add usage examples"]
    }
  },
  "recommendations": [
    {
      "priority": "high",
      "suggestion": "Add test-friendly reset mechanism or refactor to use DI",
      "rationale": "Current implementation makes testing difficult"
    },
    {
      "priority": "medium",
      "suggestion": "Document singleton rationale in JSDoc",
      "rationale": "Team members should understand why global state is necessary here"
    }
  ]
}
```

## 约束与指南

### 只读分析
- **禁止修改**：本技能只分析和建议，不修改代码
- **禁止创建文件**：不生成重构后的代码文件
- **用户决策**：所有建议均需用户明确批准后方可实施

### 语言范围
- **主要支持**：TypeScript（`.ts`、`.tsx`）
- **次要支持**：JavaScript（`.js`、`.jsx`）
- **不支持**：其他语言（Python、Java、C#）

### 模式覆盖
- **创建型（5）**：单例、工厂方法、抽象工厂、建造者、原型
- **结构型（7）**：适配器、桥接、组合、装饰器、外观、享元、代理
- **行为型（11）**：职责链、命令、迭代器、中介者、备忘录、观察者、状态、策略、模板方法、访问者、解释器

### 性能考量
- **大型代码库（>500 个文件）**：使用 `--scope` 限制扫描范围到特定目录
- **并行搜索**：每种模式的 Grep 搜索独立运行
- **缓存**：技术栈检测结果在会话内缓存，避免重复读取 package.json

## 使用示例

### 基本检测
```bash
# 检测 src/ 中的所有模式
/design-patterns detect src/

# 仅检测创建型模式
/design-patterns detect src/ --category=creational

# 专注于特定模式
/design-patterns detect src/ --pattern=singleton
```

### 针对性建议
```bash
# 获取支付模块的建议
/design-patterns suggest src/payment/

# 专注于特定代码异味
/design-patterns suggest src/ --smell=switch-on-type

# 仅高优先级建议
/design-patterns suggest src/ --priority=high
```

### 质量评估
```bash
# 评估特定文件
/design-patterns evaluate src/services/api-client.ts

# 评估所有单例
/design-patterns evaluate src/ --pattern=singleton

# 完整质量报告
/design-patterns evaluate src/ --detailed
```

## 与其他技能的集成

本技能可被以下技能继承：
- `refactoring-specialist.md` → 为重构提供模式知识
- `code-reviewer.md` → 在审查流程中加入模式检测
- `architecture-advisor.md` → 用模式使用情况辅助架构决策

## 参考文件

- `reference/patterns-index.yaml` → 23 种模式的机器可读索引及元数据
- `reference/creational.md` → 创建型模式文档
- `reference/structural.md` → 结构型模式文档
- `reference/behavioral.md` → 行为型模式文档
- `signatures/detection-rules.yaml` → 用于检测的正则表达式模式和启发式规则
- `signatures/code-smells.yaml` → 代码异味到适用模式的映射
- `signatures/stack-patterns.yaml` → 技术栈检测规则及原生模式等效方案
- `checklists/pattern-evaluation.md` → 质量评估标准与评分指南

## 版本信息

**技能版本**：1.0.0
**模式覆盖**：23 种 GoF 模式
**支持技术栈**：8 种（React、Angular、NestJS、Vue、Express、RxJS、Redux/Zustand、ORM）
**最后更新**：2026-01-21
