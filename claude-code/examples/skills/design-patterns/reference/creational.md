> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "创建型设计模式"
description: "单例、工厂、建造者、原型等对象创建模式参考"
tags: [reference, design-patterns, architecture]
---

# 创建型设计模式

处理对象创建机制的模式，旨在以适合当前场景的方式创建对象。

## 单例

### 定义
确保一个类只有一个实例，并提供对该实例的全局访问点。

### 适用场景
- [x] 需要一个类恰好只有一个实例（配置、日志、数据库连接）
- [x] 需要对单个对象进行受控访问
- [x] 实例应支持通过子类进行扩展

**警告**：该模式常被过度使用。请优先考虑依赖注入或基于上下文的替代方案。

### TypeScript 签名
```typescript
class Singleton {
  private static instance: Singleton;
  private constructor() {
    // 私有构造函数防止外部实例化
  }

  public static getInstance(): Singleton {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }

  public someMethod(): void {
    // 业务逻辑
  }
}

// 使用方式
const instance = Singleton.getInstance();
```

### 技术栈原生替代方案

**React**:
```typescript
// 用 Context 代替单例
const ConfigContext = createContext<Config>(defaultConfig);

export const ConfigProvider = ({ children }: Props) => {
  const [config] = useState(() => loadConfig());
  return <ConfigContext.Provider value={config}>{children}</ConfigContext.Provider>;
};
```

**Angular**:
```typescript
// 可注入服务（默认为单例）
@Injectable({ providedIn: 'root' })
export class ConfigService {
  // 通过 DI 自动实现单例
}
```

**NestJS**:
```typescript
@Injectable() // 默认作用域为 SINGLETON
export class AppService {}
```

### 识别标志
- `private constructor`
- `static getInstance()` 方法
- `private static instance` 字段
- 懒初始化检查：`if (!instance)`

### 解决的代码异味
- **全局状态访问**：提供受控访问，替代分散的全局变量
- **共享资源的多个实例**：确保单一数据库连接、配置对象等

### 常见错误
- **难以测试**：静态方法和全局状态使单元测试困难
  - *解决方案*：改用依赖注入，或为测试提供 `resetInstance()` 方法
- **线程安全问题**：（在 JavaScript 单线程模型中不太相关，但对 Node.js workers 很重要）
- **隐式依赖**：使用 `getInstance()` 的类存在隐式耦合
- **违反单一职责**：通常同时管理实例创建和业务逻辑

### 评估标准
- **可测试性**：3/10（难以 mock，存在全局状态）
- **线程安全**：7/10（在 JS 中不那么关键）
- **可扩展性**：5/10（子类化较复杂）

---

## 工厂方法

### 定义
定义一个创建对象的接口，但让子类决定实例化哪个类。

### 适用场景
- [x] 类无法预知需要创建的对象类型
- [x] 类希望由其子类来指定所创建的对象
- [x] 将对象创建逻辑集中以避免重复
- [x] 需要将对象创建与使用解耦

### TypeScript 签名
```typescript
// 产品接口
interface Product {
  operation(): string;
}

// 具体产品
class ConcreteProductA implements Product {
  operation(): string {
    return 'Product A';
  }
}

class ConcreteProductB implements Product {
  operation(): string {
    return 'Product B';
  }
}

// 创建者（工厂）
abstract class Creator {
  // 工厂方法
  abstract createProduct(): Product;

  // 使用产品的业务逻辑
  someOperation(): string {
    const product = this.createProduct();
    return `Creator: ${product.operation()}`;
  }
}

// 具体创建者
class CreatorA extends Creator {
  createProduct(): Product {
    return new ConcreteProductA();
  }
}

class CreatorB extends Creator {
  createProduct(): Product {
    return new ConcreteProductB();
  }
}

// 使用方式
const creator: Creator = new CreatorA();
console.log(creator.someOperation());
```

### 现代 TypeScript 替代方案
```typescript
// 不使用继承的更简洁方式
type ProductType = 'A' | 'B';

function createProduct(type: ProductType): Product {
  switch (type) {
    case 'A': return new ConcreteProductA();
    case 'B': return new ConcreteProductB();
  }
}
```

### 识别标志
- 方法名为 `create*()` 且返回接口或抽象类
- 基类中有 `abstract createProduct()`
- 子类重写工厂方法
- 基于类型/种类参数的 `switch` 或 `if-else`

### 解决的代码异味
- **与具体类的紧密耦合**：客户端代码依赖接口而非实现
- **实例化逻辑重复**：集中于工厂方法中
- **分散的 switch 语句**：统一集中到一处

### 常见错误
- **与简单工厂混淆**：工厂方法使用继承；简单工厂使用组合
- **参数过多**：应使用默认配置创建对象
- **忘记将工厂方法设为抽象**：失去了子类专化的意义

### 评估标准
- **可测试性**：8/10（产品易于 mock）
- **灵活性**：9/10（新增产品无需修改现有代码）
- **复杂度**：6/10（引入了继承层级）

---

## 抽象工厂

### 定义
提供一个创建一系列相关或相互依赖对象的接口，而无需指定其具体类。

### 适用场景
- [x] 系统应独立于其产品的创建方式
- [x] 系统需要配置多个产品族之一
- [x] 一组相关产品对象必须一起使用
- [x] 希望提供产品库且只暴露接口

### TypeScript 签名
```typescript
// 抽象产品
interface AbstractProductA {
  usefulFunctionA(): string;
}

interface AbstractProductB {
  usefulFunctionB(): string;
  anotherFunctionB(collaborator: AbstractProductA): string;
}

// 具体产品 - 第一族
class ConcreteProductA1 implements AbstractProductA {
  usefulFunctionA(): string {
    return 'Product A1';
  }
}

class ConcreteProductB1 implements AbstractProductB {
  usefulFunctionB(): string {
    return 'Product B1';
  }

  anotherFunctionB(collaborator: AbstractProductA): string {
    return `B1 collaborating with ${collaborator.usefulFunctionA()}`;
  }
}

// 具体产品 - 第二族
class ConcreteProductA2 implements AbstractProductA {
  usefulFunctionA(): string {
    return 'Product A2';
  }
}

class ConcreteProductB2 implements AbstractProductB {
  usefulFunctionB(): string {
    return 'Product B2';
  }

  anotherFunctionB(collaborator: AbstractProductA): string {
    return `B2 collaborating with ${collaborator.usefulFunctionA()}`;
  }
}

// 抽象工厂
interface AbstractFactory {
  createProductA(): AbstractProductA;
  createProductB(): AbstractProductB;
}

// 具体工厂
class ConcreteFactory1 implements AbstractFactory {
  createProductA(): AbstractProductA {
    return new ConcreteProductA1();
  }

  createProductB(): AbstractProductB {
    return new ConcreteProductB1();
  }
}

class ConcreteFactory2 implements AbstractFactory {
  createProductA(): AbstractProductA {
    return new ConcreteProductA2();
  }

  createProductB(): AbstractProductB {
    return new ConcreteProductB2();
  }
}

// 客户端代码
function clientCode(factory: AbstractFactory) {
  const productA = factory.createProductA();
  const productB = factory.createProductB();

  console.log(productB.anotherFunctionB(productA));
}

// 使用方式
clientCode(new ConcreteFactory1());
clientCode(new ConcreteFactory2());
```

### 识别标志
- 工厂接口中有多个 `create*()` 方法
- 相关产品族（例如 Windows/Mac 的 Button + Checkbox）
- 工厂实现返回不同的产品族
- 接口中有 2 个以上工厂方法

### 解决的代码异味
- **不一致的产品族**：确保兼容的产品一起创建（Windows Button + Windows Checkbox，而非混用）
- **创建逻辑分散**：将相关对象的创建集中管理

### 常见错误
- **过度设计**：对简单场景往往过于复杂；工厂方法可能已足够
- **产品族不够灵活**：添加新产品类型需要修改所有工厂
- **与工厂方法混淆**：抽象工厂创建产品族；工厂方法创建单一产品类型

### 评估标准
- **可测试性**：8/10（工厂易于 mock）
- **一致性**：10/10（保证产品兼容性）
- **复杂度**：4/10（复杂度高，类较多）

---

## 建造者

### 定义
将复杂对象的构建与其表示分离，允许逐步构建。

### 适用场景
- [x] 对象有很多可选参数（超过 4 个）
- [x] 构建过程应允许不同的表示形式
- [x] 需要逐步构建复杂对象
- [x] 希望避免"伸缩构造函数"反模式

### TypeScript 签名
```typescript
// 产品
class House {
  public walls: string = '';
  public doors: number = 0;
  public windows: number = 0;
  public roof: string = '';
  public garage: boolean = false;
  public pool: boolean = false;

  public describe(): string {
    return `House with ${this.walls} walls, ${this.doors} doors, ${this.windows} windows, ${this.roof} roof, garage: ${this.garage}, pool: ${this.pool}`;
  }
}

// 建造者
class HouseBuilder {
  private house: House;

  constructor() {
    this.house = new House();
  }

  public setWalls(walls: string): this {
    this.house.walls = walls;
    return this;
  }

  public setDoors(doors: number): this {
    this.house.doors = doors;
    return this;
  }

  public setWindows(windows: number): this {
    this.house.windows = windows;
    return this;
  }

  public setRoof(roof: string): this {
    this.house.roof = roof;
    return this;
  }

  public addGarage(): this {
    this.house.garage = true;
    return this;
  }

  public addPool(): this {
    this.house.pool = true;
    return this;
  }

  public build(): House {
    const result = this.house;
    this.house = new House(); // 重置以备下次构建
    return result;
  }
}

// 使用方式
const house = new HouseBuilder()
  .setWalls('brick')
  .setDoors(2)
  .setWindows(6)
  .setRoof('tile')
  .addGarage()
  .build();
```

### 现代 TypeScript 替代方案（类型安全建造者）
```typescript
// 渐进式类型安全：每一步解锁下一步
type HouseBuilderState<
  TWalls extends boolean = false,
  TRoof extends boolean = false
> = {
  setWalls: TWalls extends true ? never : (walls: string) => HouseBuilderState<true, TRoof>;
  setRoof: TRoof extends true ? never : (roof: string) => HouseBuilderState<TWalls, true>;
  build: TWalls extends true ? (TRoof extends true ? () => House : never) : never;
};
```

### 识别标志
- 方法链（返回 `this` 或建造者类型）
- `build()` 方法返回最终产品
- `with*()` 或 `set*()` 方法
- 可选字段逐步设置

### 解决的代码异味
- **伸缩构造函数**：参数过多的构造函数
  ```typescript
  // 不好
  new House(walls, doors, windows, roof, garage, pool, garden, basement, ...);

  // 用建造者更好
  new HouseBuilder().setWalls('brick').setRoof('tile').build();
  ```
- **参数顺序不清晰**：命名方法使意图明确
- **可选参数复杂性**：建造者优雅处理可选特性

### 常见错误
- **可变建造者**：复用建造者可能导致意外状态
  - *解决方案*：在 `build()` 后重置内部状态
- **不完整的建造者**：在 `build()` 中未验证必填字段
  - *解决方案*：使用 TypeScript 类型强制必要步骤
- **场景过简**：如果参数少于 4 个，构造函数或对象字面量可能更简单

### 评估标准
- **可测试性**：9/10（易于创建测试固件）
- **可读性**：10/10（流式接口具有自描述性）
- **复杂度**：7/10（引入了建造者类）

---

## 原型

### 定义
通过复制现有对象（原型）来创建新对象，而非从头创建。

### 适用场景
- [x] 对象创建代价高昂（复杂初始化、数据库查询）
- [x] 需要避免仅为改变初始化而创建子类
- [x] 系统应独立于产品的创建方式
- [x] 需要在运行时指定要实例化的类

### TypeScript 签名
```typescript
// 原型接口
interface Prototype {
  clone(): Prototype;
}

// 具体原型
class ConcretePrototype implements Prototype {
  public field: number;
  public complexObject: { data: string };

  constructor(field: number, complexObject: { data: string }) {
    this.field = field;
    this.complexObject = complexObject;
  }

  // 浅克隆
  public clone(): ConcretePrototype {
    return Object.create(this);
  }

  // 深克隆
  public deepClone(): ConcretePrototype {
    return new ConcretePrototype(
      this.field,
      { data: this.complexObject.data } // 克隆嵌套对象
    );
  }
}

// 使用方式
const original = new ConcretePrototype(42, { data: 'important' });
const shallowCopy = original.clone();
const deepCopy = original.deepClone();

// 浅拷贝共享嵌套对象
shallowCopy.complexObject.data = 'modified';
console.log(original.complexObject.data); // 'modified' (!)

// 深拷贝是独立的
deepCopy.field = 99;
console.log(original.field); // 42 (不变)
```

### 现代 JavaScript 替代方案
```typescript
// 展开运算符（浅拷贝）
const copy1 = { ...original };

// Object.assign（浅拷贝）
const copy2 = Object.assign({}, original);

// structuredClone（深拷贝，现代浏览器/Node 17+）
const copy3 = structuredClone(original);

// JSON（深拷贝，但有限制：不支持函数、undefined 等）
const copy4 = JSON.parse(JSON.stringify(original));
```

### 识别标志
- `clone()` 方法
- `Object.create()`
- `structuredClone()`
- `JSON.parse(JSON.stringify())` 模式
- 展开运算符 `{ ...obj }`

### 解决的代码异味
- **初始化代价高昂**：克隆代替重新初始化
- **复杂对象图**：克隆保留对象关系
- **运行时类型指定**：克隆原型而非硬编码类型

### 常见错误
- **浅克隆与深克隆混淆**：浅克隆共享嵌套对象
  ```typescript
  // 如果修改嵌套对象会有危险
  const shallow = { ...original };
  ```
- **循环引用**：`JSON.stringify` 对循环引用会失败
  - *解决方案*：使用 `structuredClone()` 或自定义克隆逻辑
- **克隆方法/函数**：某些方式会丢失方法
  ```typescript
  JSON.parse(JSON.stringify(obj)); // 丢失所有方法！
  ```
- **未克隆私有状态**：确保所有必要状态都被复制

### 评估标准
- **性能**：9/10（比重新初始化更快）
- **简洁性**：7/10（浅克隆与深克隆较难把握）
- **可靠性**：6/10（嵌套对象容易出错）

---

## 汇总表

| 模式 | 复杂度 | 使用频率 | 主要优势 |
|------|--------|----------|----------|
| 单例 | 低 | 高 | 全局访问控制 |
| 工厂方法 | 中 | 高 | 解耦创建与使用 |
| 抽象工厂 | 高 | 中 | 一致的产品族 |
| 建造者 | 中 | 高 | 流式构建复杂对象 |
| 原型 | 低 | 低 | 高效克隆 |

## 最佳实践

1. **优先使用组合而非继承**：工厂和建造者通常优于单例
2. **使用技术栈原生替代方案**：React Context 优于单例，DI 优于 `getInstance()`
3. **充分利用 TypeScript**：使用泛型和类型约束实现类型安全的建造者
4. **测试友好设计**：避免单例；使用依赖注入
5. **简单优先**：工厂方法已够用时，不要使用抽象工厂

## 参考资料

- *Design Patterns: Elements of Reusable Object-Oriented Software*（四人帮）
- *Effective TypeScript* by Dan Vanderkam
- [Refactoring Guru: Creational Patterns](https://refactoring.guru/design-patterns/creational-patterns)
