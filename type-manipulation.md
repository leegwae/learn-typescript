# Creating Types from Types

## The `keyof` Type Operator

```typescript
keyof 객체타입
```

- `keyof` 연산자는 `객체타입`의 문자열 혹은 숫자 리터럴 키의 union을 반환한다.

```typescript
type Key = keyof { foo: number; bar: string };
// type Key = "foo" | "bar"
```

### 인덱스 시그니처와 사용할 경우

- index signature property가 `number`인 경우, `number`를 반환한다.

```typescript
type Key = keyof { [index: number]: unknown };
// type Key = number;
```

- index signature property가 `string`인 경우, `string | number`를 반환한다. 이는 객체 프로퍼티에 숫자를 지정해도 문자열로 암묵적 변환되기 때문이다.

```typescript
type Key = keyof { [index: string]: unknown };
// type Key = string | number
```

## The `typeof` type operator

```typescript
let foo = 'hello world';

// in an expression context
typeof foo;
// in a type context
let bar: typeof foo;
```

- `typeof` 연산자는 *expression* context에서는 표현식이 평가된 값의 타입을 나타내는 문자열을 반환한다(자바스크립트).
- `typeof` 연산자는 *type* context에서는 **식별자**의 타입을 반환한다.

## Indexed Access Types

```typescript
type Champion = { name: string; class: string; }
// 프로퍼티 키로 접근 > type Name = string
type Name = Champion['name'];
// union으로 접근 > type C1 = string | number
type C1 = Champion['name' | 'class'];
type C2 = Champion[keyof Champion];

type C3 = Champion['age']; // 💥 Property 'age' does not exist on type 'Champion'.
```

- 인덱스로 접근하면 타입 혹은 타입의 union을 반환한다.
- 존재하지 않는 프로퍼티를 사용하여 인덱스로 접근하면 에러를 반환한다.

```typescript
const key = 'name';
// type Name = Champion[key]; < 💥 'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?
type Name = Champion[typeof key];
```

- 이때 사용하는 프로퍼티 이름은 리터럴 값이 아니라 타입이다.

### 배열의 요소의 타입을 union으로 만들기

- 원시값의 배열인 경우, 해당 값들의 union 만들기

```typescript
const VARIABLES = ['FOO', 'BAR', 'BAZ'] as const;
type Variable = typeof VARIABLES[number];
// type Variable = "FOO" | "BAR" | "BAZ"
```

- 객체의 배열인 경우, 해당 객체들의 union 만들기

```typescript
const obj = [
  { a: 1 },
  { b: 3 }
];
type Obj = typeof obj[number];
// type Obj = { a: number; b?: undefined; } | { b: number; a?: undefined; }
```

## Conditional Types

```typescript
SomeType extends OtherType ? TrueType : FalseType;
```

### 오버로딩에 사용하기

```typescript
type Foo = { foo: boolean };
type Bar = { bar: string };
function fn(value: string): Foo;
function fn(value: number): Bar;
function fn(value: string | number): Foo | Bar {
  /* 구현 */
}
```

- 위와 같은 경우라면 제네릭과 conditional type을 함께 사용한다.

```typescript
function fn<T extends number | string>(value: T): T extends number ? Foo : Bar {
  throw 'unimplemented';
}
```

TODO: 그래서 실제로 반환 값을 어떻게 타이핑해야하는지?

### Conditional type Constraints

어떤 객체의 `value` 프로퍼티의 타입을 알아내고 싶다고 하자. indexed access type을 사용하면 된다.

```typescript
type ValueOf<T extends { value: unknown }> = T['value'];
type Atom = { value: string; };
type AtomValue = ValueOf<Atom>; // type AtomValue = string
```

객체에 `value` 프로퍼티가 없다면 `never`를 반환하도록 conditional type을 사용할 수 있다.

```typescript
type ValueOf<T> = T extends { value: unknown } ? T['value'] : never;
type Atom = { value: string; };
type AtomValue = ValueOf<Atom>; // type AtomValue = string

type Bird = { fly: () => void; };
type BirdValue = ValueOf<Bird>;	// type BirdValue = never
```

### Inferring Within Conditional Types

타입 변수가 배열 타입이면 배열 요소의 타입을, 그렇지 않다면 타입 변수를 그대로 반환하고 싶다면 indexed access type을 사용하면 된다.

```typescript
type Flatten<T> = T extends any[] ? T[number] : T;
type Num = Flatten<number[]>;	// type Num = number
```

이때 `infer` 키워드를 사용해볼 수도 있다.

```typescript
type Flatten<T> = T extends Array<infer Item> ? Item : T;
type Str = Flatten<string[]>;	// type Str = string
```

TODO: infer

### Distributive Conditional Types

- conditional type은 generic에 union이 주어지면 distributive하게 된다.

```typescript
type ToArray<T> = T extends any ? T[] : never;
type NumArrOfStrArr = ToArray<number | string>;	// type NumArrOfStrArr = string[] | number[]
```

- 이를 막으려면 대괄호로 타입 변수를 감싼다.

```typescript
type ToArray<T> = [T] extends any ? T[] : never;	// type NumArrOfStrArr = (string | number)[]
```

## Mapped Types

- Mapped Type은 index signature 문법으로 만든다.

어떤 객체 타입의 모든 프로퍼티에 대하여 값의 타입이 `boolean`인 객체 타입을 만들고 싶다고 하자.

```typescript
type Obj = { foo: number; bar: string; };
type Options = {
  [Property in keyof Obj]: boolean;
}
// type Options = { foo: boolean; bar: boolean; }
```

### Mapping Modifiers

- `readonly` modifier를 명시하면 프로퍼티를 readonly로 만든다.
- `?` modifier를 명시하면 프로퍼티를 optional로 만든다.
- 각각의 modifier 앞에 `-`를 명시하면 프로퍼티에서 해당 modifier를 제거한다. 즉, `+`가 기본값이다.

```typescript
type Obj = { 
  readonly foo: number;
  bar: string;
};
type CreateMutable<T> = {
  -readonly [P in keyof T]: T[P];
};
type Options = CreateMutable<Obj>;
// type Options = { foo: number; bar: string; }
```

### Key Remapping via `as`

4.1 이상부터 프로퍼티를 다른 타입으로 remapping할 수 있다.

```typescript
type ClickEvent = { type: 'click'; x: number; y: number; };
type ScrollEvent = { type: 'drag'; };

type EventHandler<Events extends { type: string }> = {
  [Event in Events as Event['type']]: (event: Event) => void; 
}

type EventHandlers = EventHandler<ClickEvent | ScrollEvent>;

// type EventHandlers = {
//   click: (event: ClickEvent) => void;
//   drag: (event: ScrollEvent) => void;
// }
```







## 참고

- https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
- https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
