# Narrowing

- 타입스크립트 타입 시스템의 목표는 어떠한 추가적인 작업(without bending over backwards to get type safety) 자바스크립트 코드를 타입 안전하게 작성하는 것이다. 타입스크립트가 정적 타입을 사용한 런타임 값을 분석하는 방식과 마찬가지로, 타입스크립트는 자바스크립트의 런타임 제어 흐름 구조(`if/else` 삼항 연산자, 루프 등)에 타입 분석을 오버레이한다.

```typescript
function printDate(date: string | Date) {
  // 타입스크립트는 typeof date === 'string'을 type guard라고 본다.
  if (typeof date === 'string') console.log(date.toDateString());
  else console.log(date);
}
```

- narrowing은 타입스크립트가 주어진 시점에서 값을 가장 구체적이고 가능한 타입으로 분석하기 위해 프로그램이 따를 수 있는 가능한 실행 경로를 따르는 것을 말한다. narrowing에는 type guard와 같은 특별한 검사, 할당, 타입을 더 구체적인 타입으로 refine하는 과정 등이 있다.

## `typeof` type guards

- `typoef` 반환값으로 검사하는 것은 type guard이다.
- 타입스크립트는 `typeof` 연산이 어떻게 서로 다른 값을 연산하는지 encode하기 때문에 자바스크립트의 특이점도 몇 가지 알고 있다.

```typescript
function print(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) { // 💥 'strs' is possibly 'null'.
      console.log(s);
    }
  }
}
```

### Truthiness narrowing

TODO; https://www.typescriptlang.org/docs/handbook/2/narrowing.html#truthiness-narrowing

나는 printAll 실행해봐도 에러가 안 뜬다.

## Equality narrowing

- 타입스크립트는 `switch`문, 일치/동등 비교 연산자를 narrowing에 사용한다.

```typescript
function test(x: string | number, y: string | boolean) {
  if (x == y) {
    // 타입스크립트는 x와 y의 타입을 알고,
    // 따라서 x와 y의 common type(string)도 알고 있다.
    // 여기서 x와 y의 type은 string으로 좁혀진다.
  }
}
```

## The `in` operator narrowing

- 타입스크립트는 `in` 연산자를 narrowing에 사용한다.
- 옵셔널 프로퍼티는 narrowing 한 이후에도 양 쪽에 속할 것이다. (아래 예시 참고)

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void};
type Human = { swim?: () => void; fly?: () => void; };

function move(animal: Fish | Bird | Human ) {
  if ('swim' in animal) {
    // animal: Fish | Human
  } else {
    // animal: Bird | Human
  }
}
```

## `instanceof` narrwoing

- 타입스크립트는 `instanceof` 연산자를 narrowing에 사용한다.

```typescript
fnction printDate(date: Date | string) {
  if (date instanceof Date) {
    // date: Date
  } else {
    // date: string
  }
}
```

## Assignments

- 타입스크립트는 할당 연산자에서 오른쪽 피연산자로 왼쪽 피연산자의 타입을 narrowing한다.
- 즉, 변수에 할당한 값의 타입으로 변수의 타입이 고정된다.

```typescript
let foo = Math.random() < 0.5 ? 'hello world' : 5;
// foo: number | string

foo = true; // 💥 Type 'boolean' is not assignable to type 'string | number'.
```

## Control flow analysis

- **control flow analysis**는 rechability로 코드를 분석하는 것이다. 타입스크립트는 type guard와 할당을 만날 때마다 이 분석을 사용한다.
- 변수를 분석할 때마다 control flow가 갈라지고 다시 머지되고 각 지점에서 서로 다른 타입으로 추론된다.

```typescript
let x: string | number | boolean;
x = Math.random() < 0.5;

console.log(x);	// x: boolean
if (Math.random() < 0.5) x = 'hello';	// x: string
else x = 100;	// x: number
return x;	// x: string
```

## Using type predicates to define a user-defined type guard

```typescript
파라미터이름 is 타입
```

- 사용자 정의 type guard를 정의할 때 **type predicate**라는 타입을 반환하는 함수를 정의한다.
- `파라미터이름`은 현재 함수 시그니처에서 파라미터의 이름이다.

```typescript
function isFish(animal: Fish | Bird): animal is Fish {
  return (animal as Fish).swim !== undefined;
}
```

TODO; `this`-based type guards. https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards

## Assertion functions

TODO; https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions

## Descriminated unios

```typescript
interface Shape {
  kind: 'circle' | 'square';
  radius?: number;
  sideLength?: number;
}
```

`kind`가 `'circle'`인 경우에만 `radius`가 있고, `kind`가 `'square'`인 경우메나 `sideLength`가 있다고 하고 싶다면, 차라리 아래와 같이 하는 것이 좋다.

```typescript
type Circle = {
  kind: 'circle';
  radius: number;
}

type Square = {
  kind: 'square';
  sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    // shape: Circle
    return Math.PI * shape.radius ** 2;
  }
}
```

- union을 이루는 타입이 동일한 프로퍼티를 가지고 이 프로퍼티가 리터럴 타입이면, 타입스크립트는 union을 descriminated union으로 보고 union의 멤버를 좁힐 수 있다.
- 이 예시에서 `kind`는 `Shape`의 discriminant 프로퍼티이다.
- `Circle`과 `Square`는 `kind`라는 필드를 가졌으나 실제로는 별개의 타입이고, 때문에 자바스크립트 코드에서 벗어나지 않게 타입스크립트 코드를 작성할 수 있다.

## The `never` type

- `never`은 존재해서는 안되는 타입을 나타낼 때 사용한다.

### Exhaustiveness checking

- `never` 타입의 값은 어떤 타입의 변수에든 할당할 수 있다.
- `never` 타입의 변수에는 `never` 타입의 값만 할당할 수 있다.
- `switch`문에서 `never`를 사용하여 철저히 검사할 수 있다.

```typescript
interface Triangle {
  kind: 'triangle';
  sideLength: number;
}
type Shape = Circle | Square | Triangle;
function getArea(shape: Shape) {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      // 💥 Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```

TODO; if문에서도 똑같지 않나?

## 스터디

- 다음 할당문에서 narrowing된 값을 추측하라

```typescript
var foo = Math.random() < 0.5 ? 'hello world' : 5; // foo: number | string
let bar = Math.random() < 0.5 ? 'hello world' : 5; // bar: number | string
const baz = Math.random() < 0.5 ? 'hello world' : 5; // baz: 'hello world' | 5
```

- 에러가 나는가? 난다면 어디에서 나는가?

```typescript
var foo = Math.random() < 0.5 ? 'hello world' : 5;
foo = true;	// 💥 Type 'boolean' is not assignable to type 'string | number'.
let bar = Math.random() < 0.5 ? 'hello world' : 5;
bar = true; // 💥 Type 'boolean' is not assignable to type 'string | number'.
const baz = Math.random() < 0.5 ? 'hello world' : 5;
baz = true; // 💥 Cannot assign to 'baz' because it is a constant.
```



## 참고

- [TypeScript Docs > Handbook > Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)