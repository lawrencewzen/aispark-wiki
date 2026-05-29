> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "行为型设计模式"
description: "观察者、策略、命令、职责链等行为模式参考手册"
tags: [reference, design-patterns, architecture]
---

# 行为型设计模式

关注算法以及对象间职责分配的模式，重点在于对象之间的通信方式。

## 职责链

### 定义
将请求沿处理器链传递，每个处理器决定是处理该请求还是将其传递给下一个处理器。

### 适用场景
- [x] 可能有多个对象处理请求，且处理器事先未知
- [x] 希望在不明确指定接收者的情况下发出请求
- [x] 处理器集合可动态指定
- [x] 处理顺序有要求

### TypeScript 签名
```typescript
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: string): string | null;
}

abstract class AbstractHandler implements Handler {
  private nextHandler: Handler | null = null;

  setNext(handler: Handler): Handler {
    this.nextHandler = handler;
    return handler; // Allows chaining: h1.setNext(h2).setNext(h3)
  }

  handle(request: string): string | null {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    return null;
  }
}

class ConcreteHandlerA extends AbstractHandler {
  handle(request: string): string | null {
    if (request === 'A') {
      return `HandlerA processed ${request}`;
    }
    return super.handle(request);
  }
}

class ConcreteHandlerB extends AbstractHandler {
  handle(request: string): string | null {
    if (request === 'B') {
      return `HandlerB processed ${request}`;
    }
    return super.handle(request);
  }
}

// Usage
const handlerA = new ConcreteHandlerA();
const handlerB = new ConcreteHandlerB();
handlerA.setNext(handlerB);

console.log(handlerA.handle('B')); // HandlerB processed B
```

### 框架原生替代方案

**Express 中间件**：
```typescript
app.use(authMiddleware);
app.use(loggingMiddleware);
app.use(errorMiddleware);
```

**NestJS Guards/Interceptors**：
```typescript
@UseGuards(AuthGuard, RolesGuard)
@UseInterceptors(LoggingInterceptor)
```

### 可修复的代码异味
- **与请求处理器紧耦合**：客户端无需知道由哪个处理器处理请求
- **复杂的条件判断逻辑**：每个处理器只需包含简单逻辑

---

## 命令

### 定义
将请求封装为对象，从而可以用不同的请求参数化客户端、对请求进行排队或记录日志，并支持可撤销操作。

### 适用场景
- [x] 需要用操作对对象进行参数化
- [x] 需要在不同时刻对请求进行排队、指定和执行
- [x] 支持撤销/重做操作
- [x] 记录变更以便系统崩溃后恢复

### TypeScript 签名
```typescript
// Command interface
interface Command {
  execute(): void;
  undo?(): void;
}

// Receiver
class Light {
  turnOn(): void {
    console.log('Light is on');
  }
  turnOff(): void {
    console.log('Light is off');
  }
}

// Concrete commands
class TurnOnCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOn();
  }

  undo(): void {
    this.light.turnOff();
  }
}

class TurnOffCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOff();
  }

  undo(): void {
    this.light.turnOn();
  }
}

// Invoker
class RemoteControl {
  private history: Command[] = [];

  execute(command: Command): void {
    command.execute();
    this.history.push(command);
  }

  undo(): void {
    const command = this.history.pop();
    if (command?.undo) {
      command.undo();
    }
  }
}

// Usage
const light = new Light();
const remote = new RemoteControl();

remote.execute(new TurnOnCommand(light));  // Light is on
remote.execute(new TurnOffCommand(light)); // Light is off
remote.undo();                             // Light is on
```

### 框架原生：Redux Actions
```typescript
const incrementAction = { type: 'INCREMENT', payload: 1 };
dispatch(incrementAction); // Command pattern
```

---

## 迭代器

### 定义
提供一种顺序访问集合元素的方式，而无需暴露其底层表示。

### 适用场景
- [x] 需要在不暴露内部结构的情况下访问集合内容
- [x] 支持对集合进行多次遍历
- [x] 为遍历不同结构提供统一接口

### TypeScript 签名
```typescript
// Iterator interface
interface Iterator<T> {
  next(): { value: T; done: boolean };
  hasNext(): boolean;
}

// Iterable collection
interface Iterable<T> {
  createIterator(): Iterator<T>;
}

// Concrete iterator
class ArrayIterator<T> implements Iterator<T> {
  private position = 0;

  constructor(private collection: T[]) {}

  next(): { value: T; done: boolean } {
    if (this.position < this.collection.length) {
      return { value: this.collection[this.position++], done: false };
    }
    return { value: null as any, done: true };
  }

  hasNext(): boolean {
    return this.position < this.collection.length;
  }
}

// Collection
class NumberCollection implements Iterable<number> {
  constructor(private items: number[]) {}

  createIterator(): Iterator<number> {
    return new ArrayIterator(this.items);
  }
}
```

### JavaScript 原生支持
```typescript
// Symbol.iterator
const collection = {
  items: [1, 2, 3],
  [Symbol.iterator]() {
    let index = 0;
    const items = this.items;
    return {
      next() {
        return index < items.length
          ? { value: items[index++], done: false }
          : { done: true, value: undefined };
      }
    };
  }
};

for (const item of collection) {
  console.log(item); // 1, 2, 3
}

// Generator (simpler)
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

for (const num of numberGenerator()) {
  console.log(num);
}
```

---

## 中介者

### 定义
定义一个封装一组对象交互方式的对象，通过避免对象之间的显式相互引用来促进松耦合。

### 适用场景
- [x] 一组对象之间的通信方式复杂
- [x] 对象因引用了太多其他对象而难以复用
- [x] 分布在各类中的行为应当可在不使用子类的情况下定制

### TypeScript 签名
```typescript
// Mediator interface
interface Mediator {
  notify(sender: object, event: string): void;
}

// Concrete mediator
class ConcreteMediator implements Mediator {
  private component1: Component1;
  private component2: Component2;

  constructor(c1: Component1, c2: Component2) {
    this.component1 = c1;
    this.component1.setMediator(this);
    this.component2 = c2;
    this.component2.setMediator(this);
  }

  notify(sender: object, event: string): void {
    if (event === 'A') {
      console.log('Mediator reacts to A and triggers:');
      this.component2.doC();
    }
    if (event === 'D') {
      console.log('Mediator reacts to D and triggers:');
      this.component1.doB();
    }
  }
}

// Base component
class BaseComponent {
  protected mediator: Mediator | null = null;

  setMediator(mediator: Mediator): void {
    this.mediator = mediator;
  }
}

// Concrete components
class Component1 extends BaseComponent {
  doA(): void {
    console.log('Component 1 does A');
    this.mediator?.notify(this, 'A');
  }

  doB(): void {
    console.log('Component 1 does B');
  }
}

class Component2 extends BaseComponent {
  doC(): void {
    console.log('Component 2 does C');
  }

  doD(): void {
    console.log('Component 2 does D');
    this.mediator?.notify(this, 'D');
  }
}

// Usage
const c1 = new Component1();
const c2 = new Component2();
const mediator = new ConcreteMediator(c1, c2);

c1.doA();
// Output:
// Component 1 does A
// Mediator reacts to A and triggers:
// Component 2 does C
```

### 框架原生：React Context
```typescript
const ChatContext = createContext<ChatMediator>(null!);

// Mediator as context
function ChatRoom({ children }: Props) {
  const sendMessage = (from: string, to: string, msg: string) => {
    // Mediator logic
  };

  return (
    <ChatContext.Provider value={{ sendMessage }}>
      {children}
    </ChatContext.Provider>
  );
}
```

### 可修复的代码异味
- **复杂的交互网络**：集中到中介者中管理
- **承担过多职责的上帝对象**：中介者只专注于协调

---

## 备忘录

### 定义
在不违反封装的前提下，捕获并外部化对象的内部状态，以便之后可以将对象恢复到该状态。

### 适用场景
- [x] 需要保存/恢复对象快照（撤销/重做）
- [x] 直接访问状态接口会暴露实现细节
- [x] 希望保留封装边界

### TypeScript 签名
```typescript
// Memento
class Memento {
  constructor(private state: string, private date: Date) {}

  getState(): string {
    return this.state;
  }

  getDate(): Date {
    return this.date;
  }
}

// Originator
class Editor {
  private content: string = '';

  type(text: string): void {
    this.content += text;
  }

  getContent(): string {
    return this.content;
  }

  save(): Memento {
    return new Memento(this.content, new Date());
  }

  restore(memento: Memento): void {
    this.content = memento.getState();
  }
}

// Caretaker
class History {
  private mementos: Memento[] = [];

  push(memento: Memento): void {
    this.mementos.push(memento);
  }

  pop(): Memento | undefined {
    return this.mementos.pop();
  }
}

// Usage
const editor = new Editor();
const history = new History();

editor.type('Hello ');
history.push(editor.save());

editor.type('World');
history.push(editor.save());

editor.type('!!!');
console.log(editor.getContent()); // Hello World!!!

editor.restore(history.pop()!);
console.log(editor.getContent()); // Hello World
```

### 可修复的代码异味
- **为实现撤销而暴露内部状态**：备忘录封装状态
- **复杂的撤销逻辑**：历史记录管理快照

---

## 观察者

### 定义
定义对象之间的一对多依赖关系，使得当一个对象状态改变时，所有依赖它的对象都会自动收到通知。

### 适用场景
- [x] 一个对象的变更需要同时改变其他对象（数量未知）
- [x] 对象需要在不知道通知对象是谁的情况下发出通知
- [x] 事件驱动架构
- [x] 响应式编程

### TypeScript 签名
```typescript
// Observer interface
interface Observer {
  update(subject: Subject): void;
}

// Subject
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Concrete subject
class ConcreteSubject implements Subject {
  private observers: Observer[] = [];
  private state: number = 0;

  attach(observer: Observer): void {
    if (!this.observers.includes(observer)) {
      this.observers.push(observer);
    }
  }

  detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index !== -1) {
      this.observers.splice(index, 1);
    }
  }

  notify(): void {
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  setState(state: number): void {
    this.state = state;
    this.notify();
  }

  getState(): number {
    return this.state;
  }
}

// Concrete observers
class ConcreteObserverA implements Observer {
  update(subject: ConcreteSubject): void {
    console.log(`ObserverA: State is now ${subject.getState()}`);
  }
}

class ConcreteObserverB implements Observer {
  update(subject: ConcreteSubject): void {
    console.log(`ObserverB: State is now ${subject.getState()}`);
  }
}

// Usage
const subject = new ConcreteSubject();
const observerA = new ConcreteObserverA();
const observerB = new ConcreteObserverB();

subject.attach(observerA);
subject.attach(observerB);

subject.setState(5);
// Output:
// ObserverA: State is now 5
// ObserverB: State is now 5
```

### 框架原生替代方案

**React**：
```typescript
const [value, setValue] = useState(0);
useEffect(() => {
  // Auto-notified on value change
}, [value]);
```

**RxJS**：
```typescript
const subject = new BehaviorSubject(0);
subject.subscribe(value => console.log(value));
subject.next(5); // Notifies subscribers
```

**Angular**：
```typescript
private data$ = new BehaviorSubject<Data>(initial);
getData() { return this.data$.asObservable(); }
```

### 可修复的代码异味
- **分散的通知逻辑**：集中到主题对象中管理
- **紧耦合**：观察者之间互不感知

### 常见错误
- **内存泄漏**：忘记取消订阅/解除注册
- **通知风暴**：过多更新引发级联通知
- **顺序依赖**：观察者之间应保持独立

---

## 状态

### 定义
允许对象在内部状态改变时改变其行为，看起来就像更改了对象所属的类。

### 适用场景
- [x] 对象行为取决于其状态
- [x] 操作中包含大量依赖状态的条件语句
- [x] 状态转换定义清晰

### TypeScript 签名
```typescript
// State interface
interface State {
  handle(context: Context): void;
}

// Context
class Context {
  private state: State;

  constructor(initialState: State) {
    this.state = initialState;
  }

  setState(state: State): void {
    console.log(`Context: Transitioning to ${state.constructor.name}`);
    this.state = state;
  }

  request(): void {
    this.state.handle(this);
  }
}

// Concrete states
class ConcreteStateA implements State {
  handle(context: Context): void {
    console.log('StateA handles request');
    context.setState(new ConcreteStateB());
  }
}

class ConcreteStateB implements State {
  handle(context: Context): void {
    console.log('StateB handles request');
    context.setState(new ConcreteStateA());
  }
}

// Usage
const context = new Context(new ConcreteStateA());
context.request(); // StateA handles request, transitions to StateB
context.request(); // StateB handles request, transitions to StateA
```

### 实际应用：文档状态
```typescript
interface DocumentState {
  publish(doc: Document): void;
  review(doc: Document): void;
}

class Draft implements DocumentState {
  publish(doc: Document): void {
    console.log('Cannot publish draft directly');
  }
  review(doc: Document): void {
    console.log('Sending for review');
    doc.setState(new InReview());
  }
}

class InReview implements DocumentState {
  publish(doc: Document): void {
    console.log('Publishing document');
    doc.setState(new Published());
  }
  review(doc: Document): void {
    console.log('Already in review');
  }
}

class Published implements DocumentState {
  publish(doc: Document): void {
    console.log('Already published');
  }
  review(doc: Document): void {
    console.log('Cannot review published document');
  }
}

class Document {
  private state: DocumentState = new Draft();

  setState(state: DocumentState): void {
    this.state = state;
  }

  publish(): void {
    this.state.publish(this);
  }

  review(): void {
    this.state.review(this);
  }
}
```

### 框架原生：React useReducer
```typescript
const reducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'DRAFT': return { status: 'draft' };
    case 'REVIEW': return { status: 'review' };
    case 'PUBLISHED': return { status: 'published' };
  }
};

const [state, dispatch] = useReducer(reducer, { status: 'draft' });
```

### 可修复的代码异味
- **基于状态的复杂条件判断**：每个状态是独立的类
- **分散的状态相关行为**：集中到各状态类中

---

## 策略

### 定义
定义一组算法，将每个算法封装起来并使其可以互换，让算法独立于使用它的客户端而变化。

### 适用场景
- [x] 多个相关类仅在行为上有所不同
- [x] 需要同一算法的不同变体
- [x] 算法使用了客户端不应知道的数据
- [x] 类中包含多个用于选择行为的条件语句

### TypeScript 签名
```typescript
// Strategy interface
interface Strategy {
  execute(a: number, b: number): number;
}

// Concrete strategies
class AddStrategy implements Strategy {
  execute(a: number, b: number): number {
    return a + b;
  }
}

class MultiplyStrategy implements Strategy {
  execute(a: number, b: number): number {
    return a * b;
  }
}

// Context
class Calculator {
  constructor(private strategy: Strategy) {}

  setStrategy(strategy: Strategy): void {
    this.strategy = strategy;
  }

  calculate(a: number, b: number): number {
    return this.strategy.execute(a, b);
  }
}

// Usage
const calculator = new Calculator(new AddStrategy());
console.log(calculator.calculate(5, 3)); // 8

calculator.setStrategy(new MultiplyStrategy());
console.log(calculator.calculate(5, 3)); // 15
```

### 框架原生：React Hooks
```typescript
// Strategies as hooks
const useCreditPayment = () => ({ process: async (amount) => { /* ... */ } });
const usePaypalPayment = () => ({ process: async (amount) => { /* ... */ } });

const usePaymentStrategy = (type: PaymentType) => {
  const strategies = {
    credit: useCreditPayment(),
    paypal: usePaypalPayment(),
  };
  return strategies[type];
};

// Usage in component
const PaymentForm = ({ type }: Props) => {
  const strategy = usePaymentStrategy(type);
  const handlePay = () => strategy.process(amount);
};
```

### 可修复的代码异味
- **按类型 switch**：`switch (type) { case 'A': ... case 'B': ... }`
  → 替换为策略选择
- **硬编码算法**：策略可以互换替换

### 常见错误
- **策略爆炸**：策略数量过多且粒度过细
- **客户端感知策略细节**：客户端不应了解策略的内部实现

---

## 模板方法

### 定义
在方法中定义算法的骨架，将某些步骤延迟到子类中实现，允许子类在不改变算法结构的前提下重新定义特定步骤。

### 适用场景
- [x] 算法的不变部分只实现一次，变化部分留给子类
- [x] 子类中的公共行为应被提取并集中管理
- [x] 控制子类的扩展方式（钩子操作）

### TypeScript 签名
```typescript
abstract class AbstractClass {
  // Template method
  templateMethod(): void {
    this.baseOperation1();
    this.requiredOperation1();
    this.baseOperation2();
    this.hook();
    this.requiredOperation2();
  }

  // Implemented operations
  baseOperation1(): void {
    console.log('AbstractClass: base operation 1');
  }

  baseOperation2(): void {
    console.log('AbstractClass: base operation 2');
  }

  // Must be implemented by subclasses
  abstract requiredOperation1(): void;
  abstract requiredOperation2(): void;

  // Hook (optional override)
  hook(): void {
    // Default empty implementation
  }
}

class ConcreteClassA extends AbstractClass {
  requiredOperation1(): void {
    console.log('ConcreteClassA: operation 1');
  }

  requiredOperation2(): void {
    console.log('ConcreteClassA: operation 2');
  }

  hook(): void {
    console.log('ConcreteClassA: hook override');
  }
}

class ConcreteClassB extends AbstractClass {
  requiredOperation1(): void {
    console.log('ConcreteClassB: operation 1');
  }

  requiredOperation2(): void {
    console.log('ConcreteClassB: operation 2');
  }
}

// Usage
const classA = new ConcreteClassA();
classA.templateMethod();
```

### 可修复的代码异味
- **重复的算法结构**：模板定义公共步骤
- **步骤顺序不一致**：模板强制规定顺序

---

## 访问者

### 定义
表示对对象结构中各元素执行的操作，允许在不修改元素类的前提下定义新操作。

### 适用场景
- [x] 对象结构包含许多接口各异的类
- [x] 需要对对象执行多种不同的操作
- [x] 对象结构很少变化，但对其的操作经常变化

### TypeScript 签名
```typescript
// Element interface
interface Element {
  accept(visitor: Visitor): void;
}

// Concrete elements
class ConcreteElementA implements Element {
  accept(visitor: Visitor): void {
    visitor.visitConcreteElementA(this);
  }

  operationA(): string {
    return 'A';
  }
}

class ConcreteElementB implements Element {
  accept(visitor: Visitor): void {
    visitor.visitConcreteElementB(this);
  }

  operationB(): string {
    return 'B';
  }
}

// Visitor interface
interface Visitor {
  visitConcreteElementA(element: ConcreteElementA): void;
  visitConcreteElementB(element: ConcreteElementB): void;
}

// Concrete visitor
class ConcreteVisitor implements Visitor {
  visitConcreteElementA(element: ConcreteElementA): void {
    console.log(`Visiting A: ${element.operationA()}`);
  }

  visitConcreteElementB(element: ConcreteElementB): void {
    console.log(`Visiting B: ${element.operationB()}`);
  }
}

// Usage
const elements: Element[] = [
  new ConcreteElementA(),
  new ConcreteElementB(),
];

const visitor = new ConcreteVisitor();
for (const element of elements) {
  element.accept(visitor);
}
```

### 可修复的代码异味
- **添加新操作需要修改元素类**：访问者将操作外部化
- **操作分散在各类中**：访问者将相关操作集中管理

### 常见错误
- **添加新元素类型**：需要修改所有访问者（扩展性差）
- **破坏封装**：访问者可能需要访问内部状态

---

## 解释器

### 定义
为某种语言定义语法表示，并提供一个使用该表示来解释语言中句子的解释器。

### 适用场景
- [x] 语法简单（复杂语法应使用解析器生成器）
- [x] 性能要求不高
- [x] 构建简单的领域特定语言（DSL）

### TypeScript 签名
```typescript
// Context
class Context {
  constructor(public input: string) {}
}

// Abstract expression
interface Expression {
  interpret(context: Context): number;
}

// Terminal expression
class NumberExpression implements Expression {
  constructor(private value: number) {}

  interpret(context: Context): number {
    return this.value;
  }
}

// Non-terminal expressions
class AddExpression implements Expression {
  constructor(private left: Expression, private right: Expression) {}

  interpret(context: Context): number {
    return this.left.interpret(context) + this.right.interpret(context);
  }
}

class MultiplyExpression implements Expression {
  constructor(private left: Expression, private right: Expression) {}

  interpret(context: Context): number {
    return this.left.interpret(context) * this.right.interpret(context);
  }
}

// Usage: (5 + 3) * 2
const context = new Context('(5 + 3) * 2');
const expression = new MultiplyExpression(
  new AddExpression(
    new NumberExpression(5),
    new NumberExpression(3)
  ),
  new NumberExpression(2)
);

console.log(expression.interpret(context)); // 16
```

### 可修复的代码异味
- **复杂的解析逻辑**：语法规则以显式类表达
- **硬编码的语言解释**：可扩展的语法结构

---

## 汇总表

| 模式 | 复杂度 | 使用频率 | 主要优势 |
|------|--------|----------|----------|
| 职责链 | 中 | 中 | 解耦发送者与接收者 |
| 命令 | 中 | 中 | 参数化、排队、撤销操作 |
| 迭代器 | 低 | 高 | 顺序访问而不暴露内部结构 |
| 中介者 | 中 | 中 | 降低对象间耦合 |
| 备忘录 | 中 | 低 | 保存/恢复状态 |
| 观察者 | 低 | 很高 | 一对多通知 |
| 状态 | 中 | 中 | 状态相关行为 |
| 策略 | 低 | 高 | 可互换的算法 |
| 模板方法 | 中 | 中 | 带变体的算法骨架 |
| 访问者 | 高 | 低 | 对对象结构执行操作 |
| 解释器 | 高 | 很低 | 简单 DSL 解释 |

## 最佳实践

1. **观察者**：务必取消订阅以防止内存泄漏
2. **策略 vs 状态**：策略从外部改变行为；状态从内部改变行为
3. **使用框架模式**：React hooks、RxJS、Redux 已内置这些模式
4. **命令用于撤销**：保存命令对象的历史记录
5. **职责链**：保持处理器逻辑简单，确保请求得到处理

## 参考资料

- *Design Patterns: Elements of Reusable Object-Oriented Software*（四人帮）
- [Refactoring Guru：行为型模式](https://refactoring.guru/design-patterns/behavioral-patterns)
- [RxJS 文档](https://rxjs.dev/)
