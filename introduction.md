# Introduction

- 자바스크립트 애플리케이션의 규모가 지수적으로 증가하고 있다.
- 타입스크립트는 자바스크립트를 위한 정적 타입 체커이다. 런타임 이전에 타입을 검사하고 보장한다.
- 타입스크립트 type annotation은 프로그램의 런타임 동작을 변경하지 않는다.



## `tsc` 시작하기

`tsc`는 타입스크립트 컴파일러이다.

### `tsc` 설치하기

```bash
npm install -g typescript
```

### `tsc`로 컴파일하기

```bash
tsc hello.ts
```

`tsc`는 `hello.ts`에 대해 다음과 같은 동작을 한다.

1. `hello.ts`를 정적 검사하여 에러를 검출한다.
2. `hello.ts`를 컴파일하여 pe annotation이 제거된 플레인 자바스크립트 파일인 `hello.js`를 emit한다.

### `tsc` 컴파일러 옵션

`tsc` 커맨드에 컴파일러 옵션들을 명시할 수 있다. 이들은 `tsconfig.json`라는 별도의 설정 파일에서 설정할 수도 있다. 아래는 대표적인 예시들이다.

- 에러가 검출되어도 자바스크립트 파일을 emit하려면 `--noEmitOnError` 컴파일러 옵션을 사용한다.

```bash
tsc --noEmitOnError hello.ts # 에러가 검출되면 자바스크립트 파일을 emit하지 않는다.
tsc --target es2015 hello.ts # 자바스크립트 파일을 원하는 버전(예: ECMAScript6)으로 emit한다.
tsc --strict hello.ts # 엄격한 검사를 적용하는 플래그(예: noImplicitAny, strictNullChecks)를 모두 활성화한다.
tsc --noImplicitAny hello.ts # 암묵적으로 any로 추론되는 변수에 오류를 발생시킨다.
tsc --strictNullChecks hello.ts # null, undefined일 수 있는 변수에 발생할 수 있는 오류를 경고한다.
```



## 참고

- [TypeScript Docs > Handbook > The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Docs > Handbook > The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)