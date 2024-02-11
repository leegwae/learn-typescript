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

## 참고

- https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
- https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
