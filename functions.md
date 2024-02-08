# Functions

## Function Type Expressions

- **function type expression**은 화살표 함수와 비슷한 문법으로 함수 타입을 선언한 것이다.

```typescript
type Fn = (arg: string) => void;
```

- 파라미터 타입을 명시하지 않으면 그 파라미터의 타입은 `any`이다.

## Call Signatures

- callable 객체는 객체 타입에 **call signature**를 작성한다.

```typescript
type DescribableFunction = {
  description: string;
  (arg: number): boolean; // call signature
}
function printWithDescription(fn: DescribableFunction, arg: number) {
  console.log(`${fn.description}(${arg}) returned ${fn(arg)}`)
}
function isEven(n: number){
  return n % 2 === 0;
}
isEven.description = 'isEven';
printWithDescription(isEven, 6);	// isEven(6) returned true
```

## Construct Signatures

- construct인 callable 객체는 객체 타입에 **construct signature**를 작성한다.

```typescript
type ConstructorFunction = {
  new (s: string): ConstructorFunction;
}
function fn(ctor: ConstructorFunction) {
  return new ctor('lux');
}
```

## Generic Functions

```typescript
function getFirstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}

const num = getFirstElement([1, 2, 3]);
// const num: number | undefined
```

- 보통 함수의 input 값과 output 값의 타입은 연관되어있다. 위에서 `getFirstElement`의 input `arr`의 타입에 따라 output의 타입이 결정된다. 이처럼 다양한 값을 연관짓기 위해 type parameter를 사용한다.
- function signature에 **type parameter**를 선언하여 generic을 사용한다.

### Inference

```typescript
function getFirstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}

const firstElement = getFirstElement([1, 'a', true]);
// const num: string | number | boolean | undefined
```

- 실로 `Type`을 명시하지 않아도 타입스크립트가 추론한다.
- TODO; literal inference을 사용하면 어떻게 될까?

```typescript
function getFirstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}

const arr = [1, 'a', true] as const;
const num = getFirstElement(arr);
// 💥 Argument of type 'readonly [1, "a", true]' is not assignable to parameter of type 'unknown[]'.
// The type 'readonly [1, "a", true]' is 'readonly' and cannot be assigned to the mutable type 'unknown[]'.
```

오류가 안나는 방법은 없을까?

### Constraints

- **constraint**를 사용하여 타입 파라미터가 허용할 수 있는 값의 종류를 제한한다. `extends` 절을 사용한다.

```typescript
function longest<Type extends { length: number; }>(a: Type, b: Type) {
  if (a.length >= b.length) return a;
  else return b;
}

console.log(longest('abcd', 'ab')); // 'abcd'
console.log(longest(10, 100));
// 💥 Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

- TODO; 이건 무슨 오류일까?

```typescript
function longest<Type extends { length: number; }>(a: Type, b: Type) {
  if (a.length >= b.length) return a;
  else return b;
}

console.log(longest([1, 2, 3, 4], 'ab'));
// 💥 Argument of type 'number[]' is not assignable to parameter of type '"ab"'.
```

#### constraint에 매칭되는 객체의 타입과 constraint로 제한된 타입은 같지 않다

- **constraint에 매칭되는 객체가 아니라 "같은 종류의 객체"여야한다. 무슨 뜻일까?**  `Type`이랑 `{length:number;}`는 다르다.

```typescript
function toObject<Type extends { length: number }>(length: number): Type {
  return { length };
  // 💥 Type '{ length: number; }' is not assignable to type 'Type'.
  // '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.
}
```

### Guidelines for Writing Good Generic Functions

1. 가능하면 type parameter에 constraint를 쓰지 않는다.
2. 가능한 한 적은 수의 type parameter를 사용한다.
3. type parameter가 한 곳에서만 사용된다면 정말로 필요한지 재고한다. (type parameter는 다양한 값을 연관짓기 위해 사용한다.)

## Optional Parameters

```typescript
// function fn(x?: number): void
function fn(x?: number) {
  // (parameter) x: number | undefined
}
```

- optional parameter로 명시하면 `undefind`와 union으로 추론된다.

```typescript
// function fn(x?: number): void
function fn(x = 10) {
	// (parameter) x: number
  console.log(x.toFixed());
}

fn(undefined);
```

- default 값을 전달해주는 경우 해당 값의 타입으로 추론된다. `undefined`를 명시적으로 전달해주어도 default 값이 할당된다.

## Function overloads

- 하나의 함수에 대하여 두 개 이상의 function signature를 정의한다면 이것을 overalod signature라고 한다.
- implementation signature가 overload signature와 달라도, 구현된 함수의 내부는 밖에서 "볼 수 없기" 때문에 타입스크립트는 함수 구현에서는 오류를 발생시키지 않는다. 그러나 해당 함수를 호출하는 문에서 오류를 발생시킬 것이다.

```typescript
function fn(x: string): void;
function fn() {/* 생략 */ }
fn();
// 💥 Expected 1 arguments, but got 0.
// An argument for 'x' was not provided.
```

- implementation signature는 function signature에 모두 부합하게 작성해야만 한다. 다른 언어에서처럼 함수를 여러 번 정의할 수 없다는 뜻이다.

```typescript
function fn(x: string): string;
function fn(x: number): boolean;
function fn(x: string | number) {
  if (typeof x === 'string') return x;
  return x % 2 == 0;
}
```

### Writing Good Overalods

1. 가능한 한 오버로딩보다는 union 타입을 사용한다.

### Declaring `this` in a Function

TODO; https://www.typescriptlang.org/docs/handbook/2/functions.html#declaring-this-in-a-function

## Other Types

### `void`

- `void`는 어떤 값도 반환하지 않는 함수의 반환값을 의미한다.
- 자바스크립트 함수는 어떤 값도 반환하지 않으면 암묵적으로 `undefined`를 반환하나 타입스크립트에서 `void`와 `undefined`는 같지 않다.

```typescript
function hello(): void {
  console.log('hello')
}
```

### `object`

- `object`는 primitive가 아닌 모든 값을 의미한다.
- `object`는 빈 객체 타입인 `{}`과 global type `Object`와 다르다. (TODO; 그러면 global type `Object`는 무엇을 의미하는가?)

```typescript
function fn(obj: Object) {}
fn({});	// TODO; 오류가 나지는 않음
fn({ name: 1 });
```

### `unknown`

- `unknown`은 *모든* 타입의 값을 의미한다.

#### `any`와의 차이점

- `any`와 `unknown`은 *모든* 타입을 의미한다. `any`와 `unknown`으로 선언된 변수에는 어떠한 타입의 값이든 할당할 수 있다.
- `any` 타입의 값은 어떤 타입의 변수에도 할당할 수 있다. `any`는 타입 체킹을 하지 않기 때문이다.

```typescript
function fn(x: any) {
    const a: number = x;
}
```

- `unknown` 타입의 값은 `any`나 `unknown` 타입의 변수에만 할당할 수 있다. 다른 타입의 변수에 할당하려면 type aseertion이나 type guard를 사용해야만 한다.

> 즉, `any`는 타입 체킹을 하지 않으므로 type aseertion이나 type guard를 강제하려면 `unknown`을 사용한다.

### `never`

- `never`은 절대 나타날 수 없는 값을 의미한다.
- `never`는 절대 값을 반환하지 않는 함수의 반환값을 의미한다.

```typescript
function fail(): never  {
  throw new Error();
}
```

### `Function`

- global type `Function`

TODO; https://www.typescriptlang.org/docs/handbook/2/functions.html#function

## Rest Parameters and Arguments

### Rest Parameters

- Rest Parameter는 타입을 명시하지 않으면 암묵적으로 `any[]`이다.
- Rest Parameter는 `Array<T>`나 `T[]`, 혹은 tuple 타입이다.

```typescript
function add(...args: number[]) {
  return args.reduce((acc, cur) => acc + cur, 0);
}

function push(...args: [string, ...boolean[]]) {}
push('a', true, false);
push('a', 'b');	// 💥 Argument of type 'string' is not assignable to parameter of type 'boolean'.
```

### Rest Arguments

- 타입스크립트는 배열을 immutable로 보지 않으므로, rest argument를 사용할 때 아래와 같은 오류가 발생할 수 있다.

```typescript
function add(a: number, b: number) {
  return a + b;
}

const op = [1, 2];
add(...op);	// 💥 A spread argument must either have a tuple type or be passed to a rest parameter.
```

이러한 경우 `as const`로 리터럴 타입을 만들거나 (변경될 리 없는) 리터럴을 넘겨 해결할 수 있다.

```typescript
const op = [1, 2] as const;
add(...op);
add(...[1, 2]);
```

## Parameter Desctructuring

```typescript
function sum({ a, b }: { a: number; b: number }) {
  return a + b;
}
```

## Assignability of Functions

### return type `void`

- 반환값이 `void`일 때 contextual typing은 함수가 어떤 값도 반환하지 않도록 강제하지 않는다.

```typescript
type voidFn = () => void;
const fn: voidFn = () => true;
const val = fn();
// const val: void
```

```typescript
type voidFn = () => void;
type Props = {
    onClick: voidFn;
}
function Component(props: Props) {}
function handleClick(): boolean { return true; }
Component({
    onClick: handleClick
})
```

- 리터럴 함수 선언에서 반환값이 `void`로 명시된 경우에는 어떤 값도 반환하지 않아야한다.

```typescript
function fn(): void {
  return true;
}
// Type 'boolean' is not assignable to type 'void'.
```

TODO; 왜 이런 동작이 존재하는 거임

- [v2 handbook](https://www.typescriptlang.org/docs/handbook/2/functions.html#void)
- [FAQ - “Why are functions returning non-void assignable to function returning void?”](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)



## 스터디

- `any`와 `unknown`의 공통점과 차이점, 언제 `any`를 사용하고 언제 `unknown`을 사용할 것인지.

### 함수 내부에서 파라미터는 어떤 타입으로 추론되는가, 또한 에러가 발생하는가?

```typescript
function fn(x?: number) {
  // (parameter) x: number | undefined
}
fn();
```

```typescript
function fn(x = 10) {
  // (parameter) x: number
  console.log(x);
}
fn(undefined); // 오류 발생 안 함
```

### reduce 타이핑하기

```typescript
interface Arr {
  age: number;
  name: string;
}

const arr: Arr[] = [
  {
    age: 3,
    name: "jayden"
  }
]

// 1. 결과 객체에 프로퍼티가 많지 않은 경우
arr.reduce((prev: { names: string[]}, curr) => {
  return {
    names: [ ...prev.names, curr.name ]
  };
}, { names: []});


// 2. 결과 객체에 프로퍼티가 많은 경우: 인덱스 시그니처 사용
arr.reduce((prev: {[k: string]: string[]}, curr) => {
  return {
    names: [ ...prev.names, curr.name ]
  };
}, {});
```



## 참고

- [TypeScript Docs > Handbook > More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)

