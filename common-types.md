# Common Types

## Type Annotations on Variables

- Type Anonotaion은 명시적으로 타입스크립트에게 변수의 타입을 알려주는 것이다.
- Type Annotation은 런타임에 제거된다.
- 자바스크립트의 타입과 타입스크립트의 type annotation이 일대일대응하는 경우도 있다.
- 타입스크립트가 타입을 자동적으로 추론할 수 있는 경우는 type annotation을 명시하지 않아도 된다. 

```typescript
const myName: string = 'lux'; // 굳이 명시하지 않아도 된다.
const myName = 'lux'; // ✅ 타입스크립트가 string으로 추론한다.
```

## The Primitives

아래 원시 타입의 값은 자바스크립트와 동일하게 나타낸다.

- `string`
- `number`
- `boolean`
- `null`
- `undefined`

### `null`, `undefined`

`null`과 `undefined`는 `strictNullChecks` 컴파일 옵션의 값에 따라 다르게 동작한다.

- `strictNullChecks`를 비활성화한 경우,
  - `null`이나 `undefined`가 될 수 있는 값은 평소와 같이 접근될 수 있다.
  - `null`이나 `undefined` 값은 어떤 타입의 변수에도 할당될 수 있다.
- `strictNullChecks`를 활성화한 경우,
  - `null`이나 `undefined` 값을 사용할 때 일종의 검사를 필요로 한다. narrowing을 통해 값을 검사할 수 있다.

```typescript
function foo(x: string | null) {
  if (x === null) { /* 생략 */ }
  else console.log(x.toUpperCase());
}
```

### `binint`

```typescript
const onHundred: bigint = BingInt(100);
```

TODO; https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#bigint

### `symbol`

TODO; https://www.typescriptlang.org/docs/handbook/symbols.html

## Arrays

- 모든 요소의 타입이 같은 배열은 `T[]`이나 `Array<T>`로 나타낸다.

```typescript
const foo: number[] = [];
const bar: Array<number> = [];
```

### `any`

- 타입스크립트는 `any` 타입의 값에는 어떤 타입 검사도 실행하지 않는다.

```typescript
const obj: any = {};
obj.foo();	// 어떤 에러도 발생하지 않는다.
```

- 타입을 명시하지 않거나, 타입스크립트가 문맥에서 타입을 추론할 수 없는 경우 컴파일러는 그 타입을 `any`로 사용한다. 이를 막으려면 `noImplicitAny` 플래그를 활성화한다.

## Functions

- 타입스크립트에서는 함수의 입력과 출력 값을 명시할 수 있다.

### Parameter Type Annotations

```typescript
function hello(name: string) {
  console.log(`Hello, ${name}!`);
}

hello(24);	// 💥 Argument of type 'number' is not assignable to parameter of type 'string'.
```

- 파라미터에 타입 어노테이션을 명시하지 않아도 타입스크립트는 인자의 개수는 검사한다.

### Return Type Annotations

- 아래와 같이 타입스크립트가 타입을 추론할 수 있는 경우 명시하지 않아도 된다.

```typescript
function getNumber(): number {
  return 24;
}
```

- 함수가 프로미스를 반환하면 `Promise` 타입을 사용해야한다.

```typescript
async function getNumber(): Promise<number> {
  return Promise.resolve(24);
}
```

### contextual typing

- contextual typing은 함수가 위치한 문맥(context)을 통해 어떤 타입이어야하는지 추론하는 것이다.
- 익명 함수는 contextual typing을 통해 보통 파라미터의 타입을 명시하지 않아도 된다.

```typescript
['lux', 'azir'].forEach(name => {
  console.log(name);
});
```

## Object Types

- `{}`안에 `프로퍼티이름: 타입`을 명시하고 프로퍼티들은 `;`나 `,`로 구분한다. (TODO; `,`가 되나?)

```typescript
function printCoord(pt: { x: number; y: number }) { /* */ }
```

### Optional Properties

- 프로퍼티를 optional로 명시하려면 `프로퍼티이름?: 타입`로 나타낸다.

```typescript
function printName(name: {first: string; last?: string}) { /* 생략 */ }
```

## Union Types

- union 타입은 타입의 합집합을 의미한다.

```typescript
function printId(id: number | string) {
  if (id === 'number') { /* id는 number이다. */ }
  else { /* id는 string으로 간주된다. */}
}
```

## Type Aliases

- type alias를 사용하여 타입에 별칭을 부여할 수 있다. 타입스크립트에서 타입과 별칭은 완전히 동일하다.

```typescript
type UserInput = string;
function input(str: string): UserInput {
  return str;
}
```

## Interfaces

- interface declaration은 객체 타입에 이름을 부여하는 또다른 방법이다.

```typescript
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) { /* 생략 */ }
```

### structurally typing

TypeScript는 structurally typed type system이다. TypeScript는 값의 structure에만 신경 쓴다. 위 코드를 예시로 들자면, `pt`에 인자로 넘어온 값이  `x`와 `y`를 가지고 있기만 하다면 타입 호환한다.

```typescript
const myObj = {
  x: 1,
  y: 1,
  name: 'foo'
};

printCoord(myObj); // ✅ 문제 없음!
```

TODO; https://toss.tech/article/typescript-type-compatibility

## Type Alias vs Interface

- 타입 확장하기

```typescript
// interface는 extends 키워드를 사용한다.
interface Animal {}
interface Cat extends Animal {}

// type alias는 intersection을 사용한다.
type Animal = {}
type Cat = Animal & {}
```

- 기존 타입에 새로운 필드 추가하기

```typescript
// interface는 re-open하여 기존의 interface에 새로운 필드를 추가할 수 있다.
interface Point {
  x: number;
}

interface Foo {
  y: number;
}

// 💥 type alias는 re-open할 수 없다.
type Point = {	// [error] Duplicate identifier 'Point'.
  x: number;
}

type Point = {
  y: number;
}
```

- 원시값 별칭 부여하기

```typescript
// type alias는 원시값에 이름을 부여할 수 있다.
type Foo = string;

// 💥 interface는 객체의 형태만 정의할 수 있으며 원시값에 이름을 부여할 수 없다.
interface Bar extends string {}
// [error] An interface cannot extend a primitive type like 'string'. It can only extend other named object types.
// [error] 'extends' clause of exported interface 'Bar' has or is using private name 'string'.
```

TODO; 

1. Prior to TypeScript version 4.2, type alias names [*may* appear in error messages](https://www.typescriptlang.org/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA), sometimes in place of the equivalent anonymous type (which may or may not be desirable). Interfaces will always be named in error messages.
2. Interface names will [*always* appear in their original form](https://www.typescriptlang.org/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWY2Q-YAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA) in error messages, but *only* when they are used by name.

## Type Assertions

- type assertion은 명시적으로 타입스크립트에게 값의 타입을 알려주는 것이다. `값 as T`로 나타낸다.

```typescript
document.getElemtnById('my-canvas') as HTMLCanvasElement;
<HTMLCanvasElement>document.getElementById('my-canvas'); // TODO; .tsx에서 사용가능
```

- type aseertion은 런타임에 제거된다.
- 더 구체적이거나 덜 구체적인 타입으로 변환하는 것만 허용한다. 예를 들어 아래와 같은 변환은 불가능하다.

```typescript
const foo = 'hello' as number;
// 💥 Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
```

이러한 경우 `any`나 `unknown`으로 변환한 다음 type assertion할 순 있다.

```typescript
const foo = 'hello' as any as number;
const bar = 'world' as unknown as boolean;
```

### Non-null Assertion Operator

- `null`이나 `undefined`가 될 수 있는 값에 `!`을 붙이면 타입에서 `null`과 `undefined`를 제거한다.

```typescript
function func(x: string | null | undefined) {
  console.log(x!.toUpperCase());
}
```



## Literal Types

- 자바스크립트 `var`과 `let` 변수가 가질 수 있는 값이 바뀔 수 있고, `const`는 바뀔 수 없으므로 타입스크립트는 `const` 변수를 literal type으로 추론한다.

```typescript
let a = 'hello'; // a는 string으로 추론된다.
const b = 'world';	// b는 'world'로 추론된다.
```

- literal type으로 type annotation할 수 있다.

```typescript
let a: 'hello' = 'hello';
a = 'world';	// 💥 Type '"world"' is not assignable to type '"hello"'.
```

- `true`와 `false`는 literal type으로, `boolean` type은 내부적으로 `true | false` union에 대한 type alias이다.

### Literal Inference

객체의 프로퍼티는 literal type으로 추론되지 않는다. 프로퍼티의 값이 수정될 수 있기 때문이다. 따라서 아래와 같은 경우가 발생할 수 있다.

```typescript
function request(url: string, method: 'GET' | 'POST') { /* 생략 */ }
const req = { url: 'https://example.com', method: 'GET' };
request(req.url, req.method);
// 💥 Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

이 경우 두 가지 해결책이 있다.

- type assertion하기

```typescript
const req = { url: 'https://example.com', method: 'GET' as 'GET' };
```

- `as const` 접미사 사용하기: `as const`를 사용하면 모든 프로퍼티를 literal type으로 보장한다.

```typescript
const req = { url: 'https://example.com', method: 'GET' } as const;
```

## 참고

- [TypeScript Docs > Handbook > Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#bigint)
