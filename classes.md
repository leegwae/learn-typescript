# Classes

## Class Members

클래스 멤버(class member)의 종류는 아래와 같다.

1. property(field)
2. method

## Properties

**필드 선언(field declaration)**은 public writable property를 만든다. 필드 선언하는 방법은 아래와 같다.

```typescript
class 클래스이름 {
  // 1. type annotation을 명시하지 않은 경우: any로 추론된다.
  식별자1;
  // 2. type annotation을 명시한 경우: 해당 타입으로 추론된다.
  식별자2: 타입;
  // 3. initializer를 가지는 경우: 해당 값의 타입으로 추론된다.
  식별자3 = 값;
}
```

- type annotation은 선택적이다. type annotation을 명시하지 않은 경우 `any`로 추론된다.
- *initializer*를 가지는 경우 해당 값으로 추론된다. 클래스 인스턴스가 생성될 때 자동으로 초기화가 이루어진다.

```typescript
// @strictPropertyInitialization: false
class Champion {
  // (property) Champion.location: any
  location;
  // (property) Champion.name: string
  name: string;
  // (property) Champion.age: number
  age = 0;
}
```

### `--strictPropertyInitialization`

- `strictPropertyInitialization` 컴파일러 옵션은 constructor에서 초기화되어야하는 클래스 필드에 대하여 오류를 발생시킨다. 기본값은 true이다.

```typescript
// @strictPropertyInitialization: true
class Champion {
  name: string; // 💥 Property 'name' has no initializer and is not definitely assigned in the constructor.
}
```

- 이때, 클래스 필드는 위와 같이 초기화 함수에 의해서가 아니라 *constructor 자체에서* 초기화되어야한다. TypeScript는 constructor 내에서 호출되는 함수가 멤버를 초기화하는지 정적 분석하지 않는다. 메서드가 오버라이딩될 경우 멤버를 초기화하는데 실패할 수 있기 때문이다.

```typescript
// @strictPropertyInitialization: true
class Champion {
  name: string;

  constructor() {
    this.init(); // 💥 Property 'name' has no initializer and is not definitely assigned in the constructor.
    
    this.name = 'lux'; // ✅ 명시적인 초기화
  }

  init() {
    this.name = 'lux';
  }
}
```

- 필드가 확실히 초기화된다는 것을 보장하려면, 식별자에 *definite assignment assertion operator* `!`를 사용한다. 이는 외부 라이브러리에 의해 초기화하려는 경우 유용하다.

```typescript
class Champion {
  name!: string;
}
```

### `readonly`

- `readonly` modifier를 명시하면 constructor 외부에서 값을 할당하는 것을 방지할 수 있다.

```typescript
class Champion {
  readonly name: string = 'lux';
  
  constructor(name?: string) {
    if (name !== undefined) this.name = name;
  }
  
  setName() {
    this.name = 'azir';	// 💥 Cannot assign to 'name' because it is a read-only property.
  }
}

const c1 = new Champion();
c1.name = 'azir'; // // 💥 Cannot assign to 'name' because it is a read-only property.
```

- `readonly` modifier를 명시하고 type annotation을 사용하지 않으면, literal type을 가진다.

```typescript
class Champion {
  readonly name = 'lux';	// (property) Champion.name: "lux"
}
```

### accessors

- accessor에는 getter와 setter가 있다. 이들은 field 기반이지만, 메서드처럼 메서드 축약 표현을 사용한다.

```typescript
class Champion {
  _name = 'lux';
  
  get name() { return this._name; }
  set name(newName) { this._name = newName; }
}
```

- TypeScript는 accessor에 다음 규칙을 적용한다.
  - `get`은 있지만 `set`은 없다면, 프로퍼티는 자동적으로 `readonly`이다.
  - setter의 파라미터 타입이 명시되지 않았다면, getter의 return type으로 추론된다.
  - getter와 setter는 동일한 [member visibility](#member-visibility)를 가져야한다.

TODO: [Announcing TypeScript 4.3 > Separate Write Types on Properties](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/#separate-write-types)

### Index Signatures

- 인덱스 시그니처를 정의할 수 있다.

```typescript
class Foo {
  [key: string]: boolean | ((key: string) => boolean);
  
  check(key: string) { return this[key] as boolean; }
}
```

## Methods

메서드(method)는 클래스의 함수 프로퍼티이다.

### constructors

- constructor는 오버로딩할 수 있다.

```typescript
class Champion {
  constructor(name: string);
  constructor(age: number);
}
```

- 자식 클래스가 constructor를 명시했을 때 `super`를 호출하지 않으면 오류를 발생시킨다.

```typescript
class Base {}
class Derived extends Base {
  constructor() {} // 💥 Constructors for derived classes must contain a 'super' call.
}
```

- 자식 클래스가 constructor에서 `this`에 접근하기 전 `super`를 호출하지 않으면 오류를 발생시킨다.

```typescript
class Parent {}
class Child extends Parent {
  constructor() {
    console.log(this);	// 'super' must be called before accessing 'this' in the constructor of a derived class.
    super();
  }
}
```

### class constructor signature와 function signature의 차이점

1. constructor는 type parameter를 가질 수 없다.
2. constructor는 return type annotation을 가질 수 없다. 클래스 인스턴스 타입이 항상 반환된다.

### 프로토타입 메서드

```typescript
class Champion {
  name: string;
  constructor(name) {
    this.name = name;
  }
  
  hello() {
    console.log(`hello, ${this.name}!`);
  }
}
```

## Class Heritage

### `implements` Clasuses

```typescript
class 클래스이름 implements 인터페이스1, 인터페이스2 {}
```

- `implements` 절을 사용하여 클래스가 인터페이스를 구현하고 있는지 검사할 수 있다.

```typescript
interface Animal {
  hello(): void;
}

class Cat implements Animal {
  world() {}
  // 💥 Class 'Cat' incorrectly implements interface 'Animal'.
  // Property 'hello' is missing in type 'Cat' but required in type 'Animal'.
}
```

- `implements` 절은 단지 검사만 할 뿐, 실제 클래스의 타입을 변경하지 않는다.

```typescript
// 예1) 인터페이스에 명시한 함수와 동일한 이름의 함수를 구현해도, 인터페이스에 명시한 대로의 파라미터로 추론해주지 않는다.
interface Chekcer {
  check: (s: string) => boolean;
}

class InputChecker implements Chekcer {
  // 💥 Parameter 's' implicitly has an 'any' type.
  check(s) { return s.lenth > 0; }
}

// 예2) 인터페이스에 명시한 선택적 프로퍼티를 실제로 가지지 않는다.
interface CheckerOption {
  prefix: string;
}
function createCheckerOption(): CheckerOption {
  return { prefix: '#' };
}
interface Checker {
  option?: CheckerOption;
}

class InputChecker implements Checker {}
const ic = new InputChecker();
ic.option = createCheckerOption();	// 💥 Property 'option' does not exist on type 'InputChecker'.
```

### `extends` Clauses

```typescript
class 자식클래스 extends 부모클래스 {}
```

- `extends` 절을 사용하여 부모 클래스로부터 프로퍼티와 메서드를 상속받을 수 있다.

```typescript
class Animal {
  move() {}
}
class Cat extends Animal {}
const cat = new Cat();
cat.move();
```

- TypeScript는 자식 클래스를 부모 클래스의 subtype으로 강제한다.

```typescript
const animals: Array<Animal> = [new Cat(), new Dog()];
animals.forEach(animal => {
  animal.move();	// ✅ 문제 없음!
})
```

따라서 자식 클래스가 부모 클래스의 제약(contract)를 따르지 않은 경우 오류를 발생시킨다.

```typescript
class Base {
  foo() {}
  bar() {}
}
class Derived extends Base {
  foo(val?: string) {}	// ✅ 문제 없음!
  bar(val: string) {}
  // 💥 Property 'bar' in type 'Derived' is not assignable to the same property in base type 'Base'.
  // Type '(val: string) => void' is not assignable to type '() => void'.
  //  Target signature provides too few arguments. Expected 1 or more, but got 0.
}
```

### Type-only Field Declarations

- target이 `ES2022` 이상이거나 `useDefineForClasFields`가 `true`인 경우, 클래스 필드는 부모 클래스의 constructor가 완료된 후에 초기화되므로 부모 클래스의 값 위에 덮어쓴다. 이는 상속받은 필드를 더 정확한 타입으로 재선언하고 싶은 경우 문제가 된다. `declare`를 사용하면 런타임에 영향을 미치지 않고 TypeScript에게 더 정확한 타입을 알려줄 수 있다.

```typescript
interface Animal {}
interface Dog extends Animal {}
class Card {
  type: Animal;
  constructor(animal: Animal) {
    this.type = animal;
  }
}
class DogCard extends Card {
  declare type: Dog;
	constructor(dog: Dog) {
    super(dog);
  }
}
```

### Initialization Order

1. 부모 클래스 필드가 초기화된다.
2. 부모 클래스 constructor가 실행된다.
3. 자식 클래스 필드가 초기화된다.
4. 자식 클래스 constructor가 실행된다.

더 정확하게는, 자식 클래스의 constructor가 실행되면 `super`의 실행으로 부모 클래스의 constructor로 실행이 위임된다.

```typescript
class Base {
  val = 'base';	// (3) 부모 클래스 필드 초기화
  constructor() {	// (4) 부모 클래스의 constructor 실행
    console.log(this.val);
  }
}

class Derived1 extends Base {
  val = 'derived1';
}

class Derived2 extends Base {
  val = 'derived2';	// (5) 자식 클래스 필드 초기화
  constructor() { // (1) 자식 클래스의 constructor 실행
    super();	// (2) 부모 클래스의 constructor로 인스턴스 생성 위임
    // (6) 자식 클래스의 constructor로 복귀 및 실행
  }
}

new Derived1();	// 'base'
new Derived2();	// 'base'
```

### Inheriting Built-in Types

- 

TODO: https://www.typescriptlang.org/docs/handbook/2/classes.html#inheriting-built-in-types

## Member Visibility

visibility modifier를 사용하여 프로퍼티나 메서드가 클래스 외부의 코드에서 보일지 말지 결정할 수 있다.

1. `public`
2. `protected`
3. `private`

### `public`

기본값으로 생략 가능하다. `public` 멤버는 어디서든지 접근할 수 있다.

```typescript
class MyClass {
  public foo;
  bar;
}
```

### `protected`

`protected` 멤버는 해당 멤버가 선언된 클래스와 서브클래스 내부에서만 접근할 수 있다.

```typescript
class MyClass {
  protected foo() { console.log('protected member'); }
  public bar() { this.foo(); }
}

class MySubclass extends MyClass {
  public baz() { this.foo(); }
}

const c = new MySubclass();
c.baz();	// ✅ 문제 없음!
c.foo();	// 💥 Property 'foo' is protected and only accessible within class 'MyClass' and its subclasses.
```

- `protected` 멤버를 오버라이딩하여 visibility를 바꿀 수 있다.

```typescript
class Base {
  protected foo = 10;
}

class Derived extends Base {
  foo = 42;
}

const d = new Derived();
console.log(d.foo);	// 42
```

### `private`

`private` 멤버는 해당 멤버가 선언된 클래스 내부에서만 접근할 수 있다.

```typescript
class Base {
  private foo = 42;
}

class Derived extends Base {
  showFoo() {
    console.log(this.foo);	// 💥 Property 'foo' is private and only accessible within class 'Base'.
  }
}
```

- `private` 멤버를 오버라이딩할 수 없다.

```typescript
class Base {
  private foo = 42;
}

class Derived extends Base {
  foo = 10;
  // 💥	Class 'Derived' incorrectly extends base class 'Base'.
  // Property 'foo' is private in type 'Base' but not in type 'Derived'.
}
```

- 동일한 클래스의 인스턴스라면 다른 인스턴스의 `private` 멤버에 접근할 수 있다.

```typescript
class Foo {
  val = 42;
  public isEqual(other: Foo) {
    return other.val === this.val;
  }
}
```

#### `#`와의 차이

- `#`가 더 *강력하게 숨긴다*. 대괄효 표기법을 사용하여 접근할 수 없고 런타임 동작을 변경한다.

```typescript
/* typescript input */
class MyClass {
  private foo = 42;
  #bar = 42;
}

/* javascript ouput */
"use strict";
var _MyClass_bar;
class MyClass {
    constructor() {
        this.foo = 42;	// private modifier를 사용한 경우, 런타임 동작을 변경하지 않는다.
        _MyClass_bar.set(this, 42);	// # 를 사용한 경우, 런타임 동작을 변경한다.
    }
}
_MyClass_bar = new WeakMap();
```

### `private`과 `protected`은 런타임에 영향을 주지 않는다

- `private`과 `protected` 멤버는 타입 체킹에서만 오류를 발생시킬뿐 런타임 동작을 변경하지 않는다.

- `private`은 대괄호 표기법으로 접근할 경우 오류를 발생시키지 않는다.

```typescript
class MyClass {
  private foo = 42;
  protected bar = 101;
}

console.log(new MyClass()['foo']);	// 42
console.log(new MyClass()['bar']);	// 101
```

## Static Members

- visibility modifier를 사용할 수 있다.

```typescript
class Base {
  public static foo() {}
  protected static bar() {}
  private static baz() {}
}

class Derived extends Base {
  qux() {
    Base.bar();
    Base.baz();	// 💥 Property 'baz' is private and only accessible within class 'Base'.
  }
}
```

- `Function` 생성자의 프로퍼티와 동일한 이름의 `static` 프로퍼티는 정의할 수 없다.

```typescript
class Champion {
  static name = 'Champion';	// 💥 Static property 'name' conflicts with built-in property 'Function.name' of constructor function 'Champion'.
}
```

## `static` Blocks

`static` 블록에서 초기화 코드를 작성할 수 있다.

TODO: 언제 쓰는지, 언제 실행되는지 잘 모르겠음

## Generic Classes

```typescript
class CacheBlock<Type> {
  value: Type;
  
  constructor(value: Type) {
    this.value = value;
  }
}

const n = new CacheBlock(42);	// const n: CacheBlock<number>
```

- 함수 호출에서와 동일한 방식을 type parameter를 추론하며, 인터페이스와 동일한 방식으로 generic constraint를 사용할 수 있다.

## `this` at Runtime

### `this` parameters

- 타입스크립트에서는 `this`라는 이름의 파라미터는 특별히 간주된다. 반드시 첫번째 파라미터여야하며, 컴파일 시 제거된다.

```typescript
type SomeType = any;
function foo(this: SomeType, x: number) {}
// 💥 A 'this' parameter must be the first parameter.
function bar(x: number, this: SomeType) {}
```

- 메서드의 `this` 파라미터는 메서드가 올바르게 호출되도록 강제할 수 있다.

```typescript
class MyClass {
  name = 'MyClass';
  getName(this: MyClass) { return this.name; }
}
const c = new MyClass();
const g = c.getName;
g();	// 💥 The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
```

## `this` Types

- `this`를 parameter type annotation에 사용하면, 메서드를 호출한 인스턴스의 타입으로 추론된다. 즉, 부모 클래스의 메서드에서 파라미터의 타입을 `type`로 명시했다면, 자식 클래스의 인스턴스에서 이 메서드를 호출할 때 파라미터의 타입이 자식 클래스로 추론된다.

```typescript
class Base {
  isEqual(other: this) {}
}
class Derived extends Base {
  x = 42;
}

const base = new Base();
const derived = new Derived();
derived.isEqual(base);
// 💥 Argument of type 'Base' is not assignable to parameter of type 'Derived'.
//  Property 'x' is missing in type 'Base' but required in type 'Derived'.
```

### `this`-based type guards

TODO: https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards

## Parameter Properties

- parameter property는 constructor 파라미터에 visibility modifier를 명시한 것으로, 해당 파라미터와 동일한 이름과 visibility의 프로퍼티를 만들고 인자로 넘어온 값으로 초기화한다.

```typescript
class Params {
  constructor (public x: number, protected y: number, private z: number) {}
}

const params = new Params(1, 2, 3);
console.log(a.x);
console.log(a.y);	// 💥 Property 'y' is protected and only accessible within class 'Params' and its subclasses.
console.log(a.z);	// 💥 Property 'z' is private and only accessible within class 'Params'.
```

## `abstract` Classes and members

- abstract 클래스를 상속받은 클래스는 반드시 abstract 클래스의 abstract 메서드와 필드를 구현해야한다.

```typescript
abstract class Base {
  abstract fn(): void;
}
class Derived extends Base {}	// 💥 Non-abstract class 'Derived' does not implement all abstract members of 'Base'
```

- abstract 클래스는 인스턴스를 만들 수 없다.

```typescript
abstract class Base {
  abstract fn(): void;
}
const base = new Base();	// 💥 Cannot create an instance of an abstract class.
```

- abstract 멤버를 가지지 않는 클래스를 concrete 하다고 한다.

### Abstract Construct Signatures

- abstract 클래스를 구현한 클래스의 인스턴스를 파라미터로 받으려면 어떻게 해야할까? abstract 클래스를 지정하면 아래와 같은 문제가 발생한다.

```typescript
abstract class Base {
  print() { console.log('base'); }
}
function print(ctor: typeof Base) {
  const instance = new ctor();	// 💥 Cannot create an instance of an abstract class.
}
print(Base);	// 어쨌든 abstract 클래스가 넘어가기도 한다.
```

- 아래와 같이 construct signature를 사용한다.

```typescript
abstract class Base {
  print() { console.log('base'); }
}
class Derived extends Base {}
function print(ctor: new () => Base) {
  const instance = new ctor();
  instance.print();
}

print(Derived);
print(Base);
// 💥 Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'.
//  Cannot assign an abstract constructor type to a non-abstract constructor type.
```

## Relationships Between Classes

클래스에도 구조적 타이핑이 적용된다.

```typescript
// @strict: false
class Animal {
  name: string;
  move: () => void;
}

class Cat {
  name: string;
  move: () => void;
  meow: () => void;
}

const animal: Animal = new Cat();
```





## 스터디

- 이 프로퍼티의 타입을 구하라

```typescript
class Foo {
  readonly val = 1;
}
```

- 콘솔 출력 결과를 구하라

```typescript
class Base {
  val = 'base';
  constructor() {
    console.log(`base constructor: ${this.val}`);
  }
}

class Derived extends Base {
  val = 'derived';
  constructor() {
    super();
    console.log(`derived constructor: ${this.val}`);
  }
}

new Derived();
// "base constructor: base"
// "derived constructor: derived" 
```

