# Generics

## Type Argument Inference

- type argument inference는 전달된 인자의 타입에 기반해 자동적으로 `Type` 타입 파라미터의 값을 추론하는 것이다.

```typescript
function foo<Type>(val: Type): Type {
  return val;
}
```

## Call Signatures with Generics

- Generic 함수의 타입을 Call Signature를 사용해 정의하기

```typescript
interface GenericFoo {
  <Type>(arg: Type): Type;
}
function foo<Type>(arg: Type): Type { return arg; }
const genericFoo: GenericFoo = foo;
```

- Non-Generic으로 사용하기

```typescript
interface GenericFoo<Type> {
  (arg: Type): Type;
}

function foo<Type>(arg: Type): Type { return arg; }
const numberFoo: GenericFoo<number> = foo;
console.log(numberFoo(42)); // 42
```

## Generic Classes

```typescript
class Stack<T> {
  stack: Array<T> = [];
  add(value: T) {
    this.stack.push(value);
  }
}
const numberStack = new Stack<number>();
numberStack.push(1);
```

TODO; https://www.typescriptlang.org/docs/handbook/2/classes.html

## Generic Parameter Defaults

```typescript
class Animal {}
class Cat extends Animal {}
class Dog extends Animal
function create<T extends Animal = Cat>(animal?: T) {}
create();
create<Dog>();
```

1. type parameter는 default type이 제공되면 optional이다.
2. required type parameter는 optional type parameter 뒤에 오면 안된다.
3. type parameter의 기본 type은 constraint를 만족해야한다.
4. type argument를 명시한 경우, required type parameter에 대한 type argument만 명시하면 된다. 명시하지 않은 type parameter는 default type으로 resolve된다.
5. default type이 명시되었고 inference가 candidate를 고르지 못하면, default type이 추론된다.
6. A class or interface declaration that merges with an existing class or interface declaration may introduce a default for an existing type parameter.
7. A class or interface declaration that merges with an existing class or interface declaration may introduce a new type parameter as long as it specifies a default.

## 참고

- https://www.typescriptlang.org/docs/handbook/2/generics.html
