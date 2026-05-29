> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "结构型设计模式"
description: "适配器、装饰器、外观、代理等组合模式参考"
tags: [reference, design-patterns, architecture]
---

# 结构型设计模式

处理对象组合与实体间关系的模式，提供将对象和类组装成更大结构的方式。

## 适配器

### 定义
将一个类的接口转换为客户端期望的另一个接口，使原本不兼容的接口能够协同工作。

### 适用场景
- [x] 希望使用一个接口不兼容的现有类
- [x] 需要将具有不同接口的第三方库集成进来
- [x] 希望创建可与不相关类协作的可复用类
- [x] 旧代码必须与新系统协同工作

### TypeScript 签名
```typescript
// Target interface (what client expects)
interface Target {
  request(): string;
}

// Adaptee (existing incompatible class)
class Adaptee {
  specificRequest(): string {
    return '.eetpadA eht fo roivaheb laicepS';
  }
}

// Adapter (makes Adaptee compatible with Target)
class Adapter implements Target {
  private adaptee: Adaptee;

  constructor(adaptee: Adaptee) {
    this.adaptee = adaptee;
  }

  public request(): string {
    const result = this.adaptee.specificRequest().split('').reverse().join('');
    return `Adapter: ${result}`;
  }
}

// Client code
function clientCode(target: Target) {
  console.log(target.request());
}

// Usage
const adaptee = new Adaptee();
const adapter = new Adapter(adaptee);
clientCode(adapter);
```

### 实战示例：第三方库集成
```typescript
// Third-party library (can't modify)
class XMLDataProvider {
  getXMLData(): string {
    return '<data><item>1</item></data>';
  }
}

// Your application expects JSON
interface JSONDataProvider {
  getJSONData(): object;
}

// Adapter
class XMLToJSONAdapter implements JSONDataProvider {
  constructor(private xmlProvider: XMLDataProvider) {}

  getJSONData(): object {
    const xml = this.xmlProvider.getXMLData();
    // Convert XML to JSON (simplified)
    return { data: { item: '1' } };
  }
}

// Usage
const xmlProvider = new XMLDataProvider();
const adapter = new XMLToJSONAdapter(xmlProvider);
const data = adapter.getJSONData();
```

### 识别特征
- 类实现目标接口
- 持有对被适配者的引用
- 通过接口转换委托给被适配者
- 命名如 `*Adapter`、`*Wrapper`

### 可解决的代码坏味道
- **接口不兼容**：使旧代码或第三方代码兼容
- **接口蔓延**：单个适配器替代修改多个客户端调用

### 常见错误
- **双向适配器**：双向转换较为复杂，建议创建两个独立适配器
- **适配器链**：多个适配器串联表明存在设计问题
- **对新代码滥用**：应从一开始就设计兼容的接口

---

## 桥接

### 定义
将抽象部分与其实现部分解耦，使两者可以独立变化。

### 适用场景
- [x] 希望避免抽象与实现之间的永久绑定
- [x] 抽象和实现都应通过子类化进行扩展
- [x] 实现的变化不应影响客户端
- [x] 希望在多个对象间共享实现（类似享元）

### TypeScript 签名
```typescript
// Implementation interface
interface Implementation {
  operationImpl(): string;
}

// Concrete implementations
class ConcreteImplementationA implements Implementation {
  operationImpl(): string {
    return 'ConcreteImplementationA';
  }
}

class ConcreteImplementationB implements Implementation {
  operationImpl(): string {
    return 'ConcreteImplementationB';
  }
}

// Abstraction
class Abstraction {
  constructor(protected implementation: Implementation) {}

  public operation(): string {
    return `Abstraction: ${this.implementation.operationImpl()}`;
  }
}

// Refined abstraction
class ExtendedAbstraction extends Abstraction {
  public operation(): string {
    return `ExtendedAbstraction: ${this.implementation.operationImpl()}`;
  }
}

// Usage
const implA = new ConcreteImplementationA();
const abstraction1 = new Abstraction(implA);
console.log(abstraction1.operation());

const implB = new ConcreteImplementationB();
const abstraction2 = new ExtendedAbstraction(implB);
console.log(abstraction2.operation());
```

### 实战示例：支持多种渲染器的 UI 组件
```typescript
// Implementation: Renderers
interface Renderer {
  renderCircle(radius: number): string;
  renderSquare(side: number): string;
}

class VectorRenderer implements Renderer {
  renderCircle(radius: number): string {
    return `Drawing circle (vector) with radius ${radius}`;
  }
  renderSquare(side: number): string {
    return `Drawing square (vector) with side ${side}`;
  }
}

class RasterRenderer implements Renderer {
  renderCircle(radius: number): string {
    return `Drawing circle (pixels) with radius ${radius}`;
  }
  renderSquare(side: number): string {
    return `Drawing square (pixels) with side ${side}`;
  }
}

// Abstraction: Shapes
abstract class Shape {
  constructor(protected renderer: Renderer) {}
  abstract draw(): string;
}

class Circle extends Shape {
  constructor(renderer: Renderer, private radius: number) {
    super(renderer);
  }
  draw(): string {
    return this.renderer.renderCircle(this.radius);
  }
}

class Square extends Shape {
  constructor(renderer: Renderer, private side: number) {
    super(renderer);
  }
  draw(): string {
    return this.renderer.renderSquare(this.side);
  }
}

// Usage: Can mix any shape with any renderer
const vectorCircle = new Circle(new VectorRenderer(), 5);
const rasterSquare = new Square(new RasterRenderer(), 10);
```

### 识别特征
- 抽象持有对实现接口的引用
- 构造函数注入实现
- 两条平行的层次结构（抽象与实现）

### 常见错误
- **与适配器混淆**：桥接是设计时决策；适配器是运行时的补救
- **简单场景过度设计**：仅在两个层次结构都需要独立变化时使用

---

## 组合

### 定义
将对象组合成树形结构以表示部分-整体层次关系，使客户端能以统一方式处理单个对象和对象组合。

### 适用场景
- [x] 希望表示对象的部分-整体层次结构
- [x] 希望客户端忽略组合对象与单个对象的差异
- [x] 树形结构适合该领域（文件系统、UI 组件、组织架构图）

### TypeScript 签名
```typescript
// Component interface
interface Component {
  operation(): string;
  add?(component: Component): void;
  remove?(component: Component): void;
  getChild?(index: number): Component;
}

// Leaf (no children)
class Leaf implements Component {
  constructor(private name: string) {}

  operation(): string {
    return this.name;
  }
}

// Composite (has children)
class Composite implements Component {
  private children: Component[] = [];

  constructor(private name: string) {}

  add(component: Component): void {
    this.children.push(component);
  }

  remove(component: Component): void {
    const index = this.children.indexOf(component);
    if (index !== -1) {
      this.children.splice(index, 1);
    }
  }

  getChild(index: number): Component {
    return this.children[index];
  }

  operation(): string {
    const results = this.children.map(child => child.operation());
    return `${this.name}(${results.join(', ')})`;
  }
}

// Usage
const tree = new Composite('root');
const branch1 = new Composite('branch1');
branch1.add(new Leaf('leaf1'));
branch1.add(new Leaf('leaf2'));

const branch2 = new Composite('branch2');
branch2.add(new Leaf('leaf3'));

tree.add(branch1);
tree.add(branch2);
tree.add(new Leaf('leaf4'));

console.log(tree.operation());
// Output: root(branch1(leaf1, leaf2), branch2(leaf3), leaf4)
```

### 实战示例：文件系统
```typescript
interface FileSystemComponent {
  getName(): string;
  getSize(): number;
  print(indent: string): void;
}

class File implements FileSystemComponent {
  constructor(private name: string, private size: number) {}

  getName(): string {
    return this.name;
  }

  getSize(): number {
    return this.size;
  }

  print(indent: string): void {
    console.log(`${indent}📄 ${this.name} (${this.size} bytes)`);
  }
}

class Directory implements FileSystemComponent {
  private children: FileSystemComponent[] = [];

  constructor(private name: string) {}

  add(component: FileSystemComponent): void {
    this.children.push(component);
  }

  getName(): string {
    return this.name;
  }

  getSize(): number {
    return this.children.reduce((sum, child) => sum + child.getSize(), 0);
  }

  print(indent: string): void {
    console.log(`${indent}📁 ${this.name} (${this.getSize()} bytes)`);
    this.children.forEach(child => child.print(indent + '  '));
  }
}

// Usage
const root = new Directory('root');
const home = new Directory('home');
home.add(new File('photo.jpg', 2048));
home.add(new File('document.pdf', 4096));

const work = new Directory('work');
work.add(new File('report.docx', 8192));

root.add(home);
root.add(work);
root.print('');
```

### 识别特征
- 具有统一接口的树形结构
- 子组件的集合
- `add()`、`remove()`、`getChild()` 方法
- 递归调用操作

### 可解决的代码坏味道
- **对组合节点与叶节点的类型判断**：统一接口消除 `instanceof` 检查
- **部分与整体的差异化处理**：客户端以统一方式对待两者

### 常见错误
- **破坏统一性**：叶节点和组合节点应共享同一接口
- **子节点管理不当**：未正确处理删除操作
- **深度递归**：树结构过深时可能导致栈溢出

---

## 装饰器

### 定义
动态地为对象附加额外职责，作为子类化的灵活替代方案来扩展功能。

### 适用场景
- [x] 需要动态且透明地为对象添加职责
- [x] 职责可以被撤销
- [x] 通过子类化扩展不切实际（存在大量可能的组合）
- [x] 希望逐步添加功能

### TypeScript 签名
```typescript
// Component interface
interface Component {
  operation(): string;
}

// Concrete component
class ConcreteComponent implements Component {
  operation(): string {
    return 'ConcreteComponent';
  }
}

// Base decorator
abstract class Decorator implements Component {
  constructor(protected component: Component) {}

  operation(): string {
    return this.component.operation();
  }
}

// Concrete decorators
class DecoratorA extends Decorator {
  operation(): string {
    return `DecoratorA(${super.operation()})`;
  }
}

class DecoratorB extends Decorator {
  operation(): string {
    return `DecoratorB(${super.operation()})`;
  }
}

// Usage: Stack decorators
const simple = new ConcreteComponent();
const decorated1 = new DecoratorA(simple);
const decorated2 = new DecoratorB(decorated1);
console.log(decorated2.operation());
// Output: DecoratorB(DecoratorA(ConcreteComponent))
```

### 框架原生替代方案

**React - 高阶组件**：
```typescript
// HOC decorator
function withAuth<P extends object>(
  Component: React.ComponentType<P>
): React.ComponentType<P> {
  return (props: P) => {
    const { user } = useAuth();
    if (!user) return <Redirect to="/login" />;
    return <Component {...props} />;
  };
}

// Usage: Stack decorators
const AuthenticatedProfile = withAuth(Profile);
const AuthenticatedAdminProfile = withLogging(withAuth(Profile));
```

**NestJS - 拦截器**：
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    return next.handle().pipe(
      tap(() => console.log('After...'))
    );
  }
}

// Apply decorator
@UseInterceptors(LoggingInterceptor)
@Controller('users')
export class UsersController {}
```

### 识别特征
- 实现与被包装对象相同的接口
- 持有对被包装对象的引用
- 委托给被包装对象并附加行为
- 可以叠加使用

### 可解决的代码坏味道
- **类爆炸**：避免为每种功能组合创建子类
- **僵化的功能添加**：动态添加/移除功能

### 常见错误
- **顺序依赖**：`DecoratorA(DecoratorB(x))` ≠ `DecoratorB(DecoratorA(x))`
- **装饰器爆炸**：过多的小装饰器难以管理
- **破坏接口约定**：装饰器必须维持接口契约

---

## 外观

### 定义
为子系统中的一组接口提供统一接口，使子系统更易于使用。

### 适用场景
- [x] 希望为复杂子系统提供简单接口
- [x] 客户端与实现类之间存在大量依赖
- [x] 希望对子系统进行分层
- [x] 需要将子系统与客户端解耦

### TypeScript 签名
```typescript
// Complex subsystem classes
class SubsystemA {
  operationA(): string {
    return 'SubsystemA';
  }
}

class SubsystemB {
  operationB(): string {
    return 'SubsystemB';
  }
}

class SubsystemC {
  operationC(): string {
    return 'SubsystemC';
  }
}

// Facade
class Facade {
  private subsystemA: SubsystemA;
  private subsystemB: SubsystemB;
  private subsystemC: SubsystemC;

  constructor() {
    this.subsystemA = new SubsystemA();
    this.subsystemB = new SubsystemB();
    this.subsystemC = new SubsystemC();
  }

  // Simplified interface
  public simpleOperation(): string {
    const resultA = this.subsystemA.operationA();
    const resultB = this.subsystemB.operationB();
    const resultC = this.subsystemC.operationC();
    return `Facade coordinates: ${resultA}, ${resultB}, ${resultC}`;
  }
}

// Client code
const facade = new Facade();
console.log(facade.simpleOperation());
// Instead of:
// const a = new SubsystemA(); const b = new SubsystemB(); const c = new SubsystemC();
// a.operationA(); b.operationB(); c.operationC();
```

### 实战示例：支付处理
```typescript
// Complex subsystems
class PaymentValidator {
  validate(amount: number, card: string): boolean {
    // Complex validation logic
    return amount > 0 && card.length === 16;
  }
}

class PaymentGateway {
  charge(amount: number, card: string): string {
    return `Charged $${amount} to ${card}`;
  }
}

class NotificationService {
  sendReceipt(email: string, transactionId: string): void {
    console.log(`Receipt sent to ${email}: ${transactionId}`);
  }
}

class TransactionLogger {
  log(transaction: string): void {
    console.log(`Logged: ${transaction}`);
  }
}

// Facade
class PaymentFacade {
  private validator = new PaymentValidator();
  private gateway = new PaymentGateway();
  private notifications = new NotificationService();
  private logger = new TransactionLogger();

  processPayment(amount: number, card: string, email: string): boolean {
    // Simplified interface for complex process
    if (!this.validator.validate(amount, card)) {
      return false;
    }

    const result = this.gateway.charge(amount, card);
    this.logger.log(result);
    this.notifications.sendReceipt(email, result);
    return true;
  }
}

// Client code (simple!)
const payment = new PaymentFacade();
payment.processPayment(100, '1234567890123456', 'user@example.com');
```

### 识别特征
- 依赖多个子系统的类
- 协调子系统的简单公共方法
- 命名如 `*Facade`、`*API`、`*Service`

### 可解决的代码坏味道
- **复杂子系统使用**：客户端无需了解子系统细节
- **紧耦合**：客户端依赖外观，而非依赖众多类

### 常见错误
- **上帝外观**：外观做了过多的事；应协调而非包含业务逻辑
- **抽象泄漏**：暴露子系统细节违背了使用外观的初衷

---

## 享元

### 定义
通过将共享状态外部化，使用共享的方式高效支持大量细粒度对象。

### 适用场景
- [x] 应用程序使用大量对象
- [x] 由于对象数量庞大导致存储成本高
- [x] 大部分对象状态可以变为外部状态（外化）
- [x] 多组对象可以用相对少量的共享对象替代

### TypeScript 签名
```typescript
// Flyweight
class Flyweight {
  constructor(private sharedState: string) {}

  operation(uniqueState: string): void {
    console.log(`Flyweight: Shared (${this.sharedState}) and unique (${uniqueState}) state.`);
  }
}

// Flyweight factory
class FlyweightFactory {
  private flyweights: Map<string, Flyweight> = new Map();

  constructor(initialFlyweights: string[][]) {
    for (const state of initialFlyweights) {
      this.flyweights.set(this.getKey(state), new Flyweight(state.join('_')));
    }
  }

  private getKey(state: string[]): string {
    return state.join('_');
  }

  getFlyweight(sharedState: string[]): Flyweight {
    const key = this.getKey(sharedState);

    if (!this.flyweights.has(key)) {
      console.log('Creating new flyweight');
      this.flyweights.set(key, new Flyweight(key));
    } else {
      console.log('Reusing existing flyweight');
    }

    return this.flyweights.get(key)!;
  }

  listFlyweights(): void {
    console.log(`FlyweightFactory: ${this.flyweights.size} flyweights:`);
    for (const key of this.flyweights.keys()) {
      console.log(key);
    }
  }
}

// Usage
const factory = new FlyweightFactory([
  ['Chevrolet', 'Camaro2018', 'pink'],
  ['Mercedes Benz', 'C300', 'black'],
]);

const flyweight1 = factory.getFlyweight(['Chevrolet', 'Camaro2018', 'pink']);
flyweight1.operation('license-123');

const flyweight2 = factory.getFlyweight(['Chevrolet', 'Camaro2018', 'pink']);
flyweight2.operation('license-456'); // Reuses same flyweight
```

### 实战示例：文本编辑器字符
```typescript
// Flyweight: Character formatting (shared)
class CharacterFormat {
  constructor(
    public font: string,
    public size: number,
    public color: string
  ) {}
}

// Flyweight factory
class FormatFactory {
  private formats = new Map<string, CharacterFormat>();

  getFormat(font: string, size: number, color: string): CharacterFormat {
    const key = `${font}_${size}_${color}`;
    if (!this.formats.has(key)) {
      this.formats.set(key, new CharacterFormat(font, size, color));
    }
    return this.formats.get(key)!;
  }
}

// Character with extrinsic state
class Character {
  constructor(
    private char: string,
    private format: CharacterFormat // Shared flyweight
  ) {}

  render(position: number): string {
    return `'${this.char}' at ${position} (${this.format.font}, ${this.format.size}px, ${this.format.color})`;
  }
}

// Document
const formatFactory = new FormatFactory();
const arial12Black = formatFactory.getFormat('Arial', 12, 'black');
const arial12Red = formatFactory.getFormat('Arial', 12, 'red');

// 10,000 characters, but only 2 format objects
const characters: Character[] = [];
for (let i = 0; i < 10000; i++) {
  const format = i % 2 === 0 ? arial12Black : arial12Red;
  characters.push(new Character('A', format));
}
```

### 识别特征
- 工厂管理共享对象池
- 内部状态（共享）与外部状态（唯一）分离
- 享元的 Map/缓存

### 常见错误
- **过早优化**：仅在内存确实成为问题时使用
- **状态分离错误**：将内部状态与外部状态混淆

---

## 代理

### 定义
为另一个对象提供代理或占位符，以控制对该对象的访问。

### 适用场景
- [x] 延迟初始化（虚拟代理）：仅在需要时创建昂贵对象
- [x] 访问控制（保护代理）：控制对原始对象的访问
- [x] 远程对象的本地代表（远程代理）
- [x] 记录日志、缓存或监控访问

### TypeScript 签名
```typescript
// Subject interface
interface Subject {
  request(): void;
}

// Real subject
class RealSubject implements Subject {
  request(): void {
    console.log('RealSubject: Handling request');
  }
}

// Proxy
class Proxy implements Subject {
  private realSubject: RealSubject | null = null;

  request(): void {
    // Access control
    if (this.checkAccess()) {
      // Lazy initialization
      if (!this.realSubject) {
        this.realSubject = new RealSubject();
      }

      // Logging
      this.logAccess();

      // Delegate to real subject
      this.realSubject.request();
    }
  }

  private checkAccess(): boolean {
    console.log('Proxy: Checking access');
    return true;
  }

  private logAccess(): void {
    console.log('Proxy: Logging access time');
  }
}

// Usage
const proxy = new Proxy();
proxy.request();
// Output:
// Proxy: Checking access
// Proxy: Logging access time
// RealSubject: Handling request
```

### 现代 JavaScript Proxy
```typescript
const target = {
  message: 'Hello',
  getValue() {
    return this.message;
  }
};

const handler = {
  get(target: any, prop: string) {
    console.log(`Accessing property: ${prop}`);
    return target[prop];
  },
  set(target: any, prop: string, value: any) {
    console.log(`Setting property: ${prop} = ${value}`);
    target[prop] = value;
    return true;
  }
};

const proxy = new Proxy(target, handler);
console.log(proxy.message); // Logs: Accessing property: message
proxy.message = 'World';     // Logs: Setting property: message = World
```

### 识别特征
- 实现与真实主体相同的接口
- 持有对真实主体的引用
- 控制访问（检查、日志记录、缓存）
- 延迟初始化真实主体

### 常见错误
- **代理链**：多个代理相互嵌套
- **性能开销**：每次访问都经过代理
- **与装饰器混淆**：代理控制访问；装饰器添加行为

---

## 汇总表

| 模式 | 复杂度 | 使用频率 | 主要优势 |
|------|--------|----------|----------|
| 适配器 | 低 | 高 | 接口兼容性 |
| 桥接 | 高 | 低 | 抽象与实现解耦 |
| 组合 | 中 | 高 | 统一处理树形结构 |
| 装饰器 | 中 | 高 | 动态添加职责 |
| 外观 | 低 | 很高 | 简化子系统接口 |
| 享元 | 高 | 低 | 内存优化 |
| 代理 | 中 | 中 | 受控访问 |

## 最佳实践

1. **适配器 vs 桥接**：适配器修复不兼容问题；桥接从设计上提供灵活性
2. **装饰器 vs 代理**：装饰器添加功能；代理控制访问
3. **外观的简洁性**：应协调各子系统，而非包含业务逻辑
4. **组合的统一性**：叶节点和组合节点必须共享接口
5. **使用原生 Proxy**：JavaScript `Proxy` 对象用于动态属性访问

## 参考资料

- *Design Patterns: Elements of Reusable Object-Oriented Software*（Gang of Four）
- [MDN: Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- [Refactoring Guru: Structural Patterns](https://refactoring.guru/design-patterns/structural-patterns)
