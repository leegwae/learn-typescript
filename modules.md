# Modules

- 타입스크립트는 최상위 `import`나 `export`, `await` 를 가진 파일은 모듈로 간주한다. 그렇지 않은 파일은 스크립트로 간주한다.
- `import`나 `export`가 없어도 모듈로 간주되길 원한다면 아래와 같이 임의의 `export` 문을 작성하라.

```typescript
export {};
```

- 타입스크립트에서는 다음 세 가지 요소가 중요하다.
  - Syntax
  - Module Resolution: 모듈 이름(경로)와 디스크의 파일의 관계는 어떠한가?
  - Module Output Target: 산출된 자바스크립트 모듈은 어때야하는가?

## TypeScript Specific ES Module Syntax

### `import type`

타입을 import할 때만 사용할 수 있다.

```typescript
// @filename: champion.ts
export type Mage = {};

// @filename: index.ts
import type { Mage } from './champion.js';
```

### inline `import type`

```typescript
import { type Mage } from './champion.js';
```

TODO: https://www.typescriptlang.org/docs/handbook/2/modules.html#es-module-syntax-with-commonjs-behavior (무슨 소리인지 잘 모르곘음)

### 왜 모듈 경로가 `.js`인가요?

TODO; https://github.com/microsoft/TypeScript/issues/50152

## TypeScript's Module Resolution Options

- Module Resolution은 `import`나 `require` 문으로부터 문자열을 가져와 문자열이 가리키는 파일이 무엇인지 결정하는 프로세스이다.
- 타입스크립트에는 두 가지 Resolution 전략이 있다.
  - Classic: `module` 옵션이 `commonjs`가 아닌 경우의 기본값으로, 레거시 호환성을 위해 존재한다.
  - Node: Node.js가 CommonJS 모드에서 동작하는 방식과 동일하며, `.ts`와 `.d.ts`에 대한 검사를 포함한다.
- 모듈 전략과 관련한 TSConfig 플래그들은 다음과 같다: `moduleResultion`, `baseUrl`, `paths`, `rootDirs`

## TypeScript's Module Output Options

- 자바스크립트 산출과 관련한 컴파일러 옵션은 아래와 같다.
  - `target`: 산출할 자바스크립트의 타겟 명세 버전
  - `module`: 산출할 자바스크립트의 타겟 모듈 시스템. 
- 모듈 간의 모든 일은 모듈 로더(module loader)가 담당하며, 런타임에 모듈 로더는 모듈을 실행하기 전 모듈의 모든 의존성을 가져오고 실행할 책임을 가진다.

## TypeScript namespaces

- ESM 이전 타입스크립트가 사용하던 모듈 포맷이다.

## CommonJS and ES Modules interop

- `esModuleInterop` 플래그를 사용해 두 모듈을 호환할 수 있다.

TODO: https://www.typescriptlang.org/tsconfig#esModuleInterop

## 참고

- https://www.typescriptlang.org/docs/handbook/2/modules.html
