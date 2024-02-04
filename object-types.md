# Object Types

타입스크립트에서 **object types**로 객체를 나타낸다.

## 객체 파라미터 타이핑하기

1. anonymous
2. named by using an interface
3. named by using a type alias

### anonymous

```typescript
function printHello(person: { name: string; age: number; }) {
  console.log(`Hello, ${person.name}!`);
}
```

### interface

```typescript
interface Person {
  name: string;
  age: number;
}
function printHello(person: Person) {
  console.log(`Hello, ${person.name}!`);
}
```

### type alias

```typescript
type Person = {
  name: string;
  age: number;
}
function printHello(person: Person) {
  console.log(`Hello, ${person.name}!`);
}
```

## Property Modifiers

객체의 프로퍼티는 타입과 선택적인지 여부, 그리고 수정 여부를 명시할 수 있다.

### Optional Properties

프로퍼티 이름 뒤에 `?`를 명시하여 프로퍼티를 optional로 만든다.

```typescript
interface Person {
  firstName: string;
  lastName?: string;
}
```

#### optional과 undefined의 union은 다르다

```typescript
function fn({ foo, bar }: { foo?: number; bar: number | undefined; }) {}
fn({});
// 💥 Argument of type '{}' is not assignable to parameter of type '{ foo?: number | undefined; bar: number | undefined; }'.
//  Property 'bar' is missing in type '{}' but required in type '{ foo?: number | undefined; bar: number | undefined; }'.
```

`bar`는 `undefined`의 union이지만 required이다. `foo`도 `undefined`의 union이지만 optional이다. `?`를 프로퍼티 이름 뒤에 붙여야 optional이다.

### `readonly` Properties

`readonly` modifier를 명시하여 프로퍼티를 read-only로 만든다.

```typescript
interface Person {
  readonly name: string;
}

const p: Person = { name: 'lux' };
p.name = 'azir'; // 💥 Cannot assign to 'name' because it is a read-only property.
```

#### `readonly`와 immutable은 다르다

```typescript
interface Point {
  x: number;
  y: number;
}
interface Place {
  name: string;
  readonly coord: Point;
}

const home: Place = { name: 'home', coord: { x: 0, y: 0 } };
home.coord.x = 1;	// coord는 변경 가능하다.
home.coord = { x: 2, y: 2 };	// 💥 Cannot assign to 'coord' because it is a read-only property.
```

read-only 프로퍼티는 "바꾸는(change)" 것이 불가능할 뿐 mutable하다. 마치 `const` 변수에 할당된 객체는 그 프로퍼티에 접근하여 변경 가능하나, 새로운 객체로 재할당이 되지 않는 것과 같다.

#### aliasing으로 `readonly` 프로퍼티 바꾸기

```typescript
interface Champion {
  name: string;
}
interface ReadonlyChampion {
  readonly name: string;
}
const writableChampion: Champion = { name: 'lux' };
const readonlyChampion: ReadonlyChampion = writableChampion;	// 문제 없음!

console.log(readonlyChampion.name);	// 'lux'
writableChampion.name = 'azir';
console.log(readonlyChampion.name);	// 'azir'
```

타입스크립트는 두 객체의 타입이 compatible한지 확인할 때 프로퍼티가 `readonly`인지 검사하지 않으므로 aliasing으로 `readonly` 프로퍼티는 바꿀 수 있다.

### Index Signatures

- 프로퍼티의 이름은 알 수 없으나 프로퍼티 이름의 타입과 프로퍼티 값의 타입을 알고 있을 때 index signature를 사용한다.

```typescript
interface StringArray {
  [index: number]: string;
}
function fn(arr: StringArray) {
	const first = arr[0];	// const first: string
}
```

- index signature property에 `string`, `number`, `symbol`, template string pattern, 그리고 이들로 이루어진 union이 허용된다.

TODO; https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures

## Excess Property Checking

객체 리터럴은 변수에 할당되거나 인자로 전달될 때 **excess property checking**을 받는다. 객체 리터럴이 "target type"이 가지고 있지 않은 프로퍼티를 가지고 있다면 오류를 발생시킨다.

```typescript
interface Person {
  name: string;
}

const p1 = { name: 'p1', age: 1 };
const p2: Person = p1;	// 덕 타이핑, 문제 없음!
const p3: Person = { name: 'p1', age: 1 }; // 💥 Object literal may only specify known properties, and 'age' does not exist in type 'Person'.
```

이런 오류를 피하고 싶다면 타입 단언을 사용할 수 있지만, 가장 좋은 건 타입을 수정하는 것이다. 객체 리터럴은 "명시적"이므로, 타입이 그를 반영하도록 하는 것이 좋다.

```typescript
interface Foo {
  a: number;
  b: number;
}
function fn(foo: Foo) {}
fn({ a: 1, b: 1, c: 2 });	// 💥 Object literal may only specify known properties, and 'c' does not exist in type 'Foo'.
// 아무래도 Foo를 아래와 같이 수정하는 것이 맞을 것이다.
interface Foo {
  a: number;
  b: number;
  c: number;
  // [index: string]: number; index signature property를 추가하는 것보단 나을 것이다.
}
```

## Interfaces and Type Aliases

- [Interfaces](https://github.com/leegwae/learn-typescript/blob/main/common-types.md#interfaces)
- [Type Aliases](https://github.com/leegwae/learn-typescript/blob/main/common-types.md#type-aliases)
- [Interfaces vs Type Aliases](https://github.com/leegwae/learn-typescript/blob/main/common-types.md#type-alias-vs-interface)

## Generic Object Types

### The `Array` Type

`Array<T>`는 `T[]`와 같다.

### The `ReadonlyArray` Type

- `ReadonlyArray<T>`는 `readonly T[]`와 같다.
- `readonly`  프로퍼티와 달리, `ReadonlyArray`는 `Array`에 할당할 수 없다.

```typescript
let readonlyArray: readonly string[] = [];
let writableArray: string[] = [];

readonlyArray = writableArray;
writableArray = readonlyArray;	 // 💥 The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
```

### Tuple Types

**tuple type**은 정확히 어떤 요소를 포함해야하는지 알고 있는 `Array`이다.

```typescript
type Pair = [string, number];
```

- tuple에서 마지막 요소를 optional로 명시할 수 있다.

```typescript
type Coordinate = [number, number, number?];

function setCoordinate(coord: Coordinate) {
  const [x, y, z] = coord; // const z: number | undefined
}
```

- tuple은 rest element를 가질 수 있다. [rest parameter와 rest argument](https://github.com/leegwae/learn-typescript/blob/main/functions.md#rest-parameters-and-arguments)에 사용할 수 있다.

```typescript
type Foo = [string, number, ...boolean[]];
type Bar = [string, ...number[], boolean];
```

### `readonly` Tuple Types

`as const`로 지정한 배열은 `readonly` tuple type으로 추론된다.

```typescript
let point = [3, 4] as const;

function distance([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distance(point);
// 💥 Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
//  The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.
```

`distance`는 인자로 들어온 배열을 변경하지 않지만 mutable tuple을 예상한다. mutable tuple에 immutable tuple을 할당할 수 없기 때문에 위와 같은 오류가 발생한다. 변경되지 않으니 아예 `readonly`로 명시해주는 것이 좋다.

```typescript
let point = [3, 4] as const;

function distance([x, y]: readonly [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distance(point);
```

TODO; https://github.com/leegwae/learn-typescript/blob/main/functions.md#rest-arguments
