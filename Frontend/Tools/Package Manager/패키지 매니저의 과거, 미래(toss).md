# 패키지 매니저의 과거, 미래 (toss)

> [출처](https://toss.tech/article/27772)

- 패키지 매니저란?
- 패키지 매니저가 동작하는 세 가지 단계
- npm, pnpm, Yarn PnP에서 패키지를 설치하는 방법의 차이
- 토스가 Yarn을 선택한 이유와 앞으로의 방향성 - 번외: 브라우저에서 웹 표준으로 패키지를 관리하는 방법

[npm](https://www.npmjs.com/), [pnpm](https://pnpm.io/ko/), 그리고 [Yarn](https://yarnpkg.com/)

<br/>

## 패키지 매니저란?

JavaScript나 TypeScript를 사용하면 `require`, `import` 구문을 사용해 외부 의존성을 참조하는데, 그걸 올바르게 참조할 수 있도록 보장해주는 프로그램.

```tsx
import React from 'Users/raon0211/path/to/react/index.js';
import { sum } from 'Users/raon0211/path/to/@toss/utils/index.js';
```

JavaScript 표준인 ECMAScript에 따르면, 원래 정확한 절대 경로나 상대 경로를 통해서만 `import` 할 수 있다. Deno나 브라우저의 JavaScript 표준 문법을 보면 다 정확한 절대 경로를 사용하고 있다. 하지만 실제로 이렇게 쓰지 않는다.

```tsx
import React from 'react';
import { sum } from '@toss/utils';

sum(1, 2, 3);
```

```tsx
const _ = require('ladash');
```

하지만 이렇게 하면 문제가 생긴다. 예를 들어 `react` 가 정확히 어떤 버전인지, `@toss/utils` 가 어떤 버전인지 모호하다는 문제. 그러면 정확한 정보는 어디서 제공하나? 소스 코드보다 상위 디렉토리인 `package.json` 파일에 명시한다.

```json
{
  "dependencies": {
    "react": "^18.2.0" // react는 ≥ 18.2.0, <19 사이의 어떤 버전이든지 쓸 수 있다고 명시
  }
}
```

이렇게 명시된 의존성 정보를 통해 모든 소스 코드 파일이 특정 버전의 라이브러리를 사용할 수 있도록 보장함. `package.json` 파일에 디펜던시를 명시, `npm install` 또는 `yarn install` 을 하면 의존성 명시된 버전을 설치하게 된다. 즉, **패키지 매니저가 앞서 이야기 한 모호한 버저닝 문제를 해결해준다.**

<br/>

## 패키지 매니저가 동작하는 세 가지 단계

버저닝 문제를 해결하기 위해 실제 동작 방식을 살펴보면 꽤 복잡하다.

yarn을 실행해보자.

![image](https://github.com/user-attachments/assets/29336338-aba2-42bc-b552-f0bca049fb1b)

이렇게 `Resolution`, `Fetch`, `Link` 세 단계로 나누어 실행된다.

### Resolution 단계

> 1. 라이브러리 버전 고정
> 2. 라이브러리의 다른 의존성 확인
> 3. 라이브러리의 다른 의존성 버전 고정

resolution의 뜻은 '문제를 해결하다' 다.

첫 번째 문제는 라이브러리를 정확한 버전으로 고정하는 문제. Resolution 단계에서는 `package.json` 파일에 명시된 버전 범위에 따라 정확한 버전을 결정한다. `react: ^18.2.0` 이라면  [^이 나타내는 규칙](https://semver.org/lang/ko/)에 따라 ≥ 18.2.0, <19 사이의 어떤 버전이든 사용할 수 있음. 패키지 매니저는 저 범위를 만족하는 선에서 가능한 최신 버전을 사용하려 함. 즉, 최신 버전인 18.3.1을 선택할 수 있음.

두 번째 문제는 라이브러리가 사용하는 다른 라이브러리, 즉 의존성 문제. 예를 들어 `@toss/use-overlay` 는 `react` 를 사용하는데 `react` 도 의존성을 갖고 있다. 그래서 의존성이 또 어떤 의존성을 가지는지 확인하는 작업이 필요함.

세 번째는 그 의존성의 버전도 고정해야 함. JavaScript에서는 의존성 버전을 범위로 명시하고, 패키지 간 의존성을 가지기 때문에 똑같은 `package.json` 에 대해 사용하는 의존성 버전이 완전히 달라질 수 있음.

예를 들어, 어떤 기기에서 Next.js 13.1 버전과 React 18.1.0 버전을 사용하고, 다른 기기에서는 Next.js 13.2, React 18.2.0을 사용할 수 있는 것이다. 개발자 입장에서는 당연히 버전마다 동작이 똑같은 건데, 슬프게도 당연히 동작이 다를 수 밖에 없고 버그가 생길 수 있다. '제 PC에서는 잘 돼요' 라고 하는게 이럴 때 생기는 것이다. 이 고정 문제를 전부 해결하는게 Resolution 단계.

즉, Resolution 단계는 모든 기기에서 고정된 버전을 사용할 수 있도록 한다. 의존성 버전을 모두 고정시키고, 의존성의 의존성을 다 찾아 그 버전도 고정시키고 결과물을 `yarn.lock` 이나 `package-lock.json` 에 저장한다.

### Fetch 단계

> - 결정된 버전의 파일을 다운로드 하는 과정

Resolution의 결과로 결정된 버전을 실제 다운로드 하는 과정. `yarn.lock` 에 명시된 패지키를 네트워크를 통해 필요한 파일을 가져온다. 일반적으로 99%는 npm 레지스트리에서 받아옴.

### Link 단계

> - Resolution/Fetch 된 라이브러리를 소스 코드에서 사용할 수 있는 환경을 제공하는 과정

가장 까다롭다. npm, pnpm, PnP(Plug'n'Play) 사례를 각각 살펴보려고 함.

#### [1] npm Linker

`node_modules` 기반 Linker를 살펴보자. 모든 의존성을 그냥 `node_modules` 디렉토리 밑에 하나하나씩 쓰는게 npm Linker의 역할이다.

예를 들어 React와 TDS 모바일 라이브러리를 사용하면, `my-service` 의 `node_modules` 하위에 React와 TDS 모바일 패키지를 추가한다. TDS 모바일 패키지에도 `node_modules` 가 있다면 `@radix-ui/dialog` 를 그 밑에 깔아주는게 npm Linker가 하는 일.

```tsx
my-service/
└─ node_modules/
|  ├─ react/
|  |  
|  └─ @tossteam/tds-mobile/
|     └─ node_modules/
|         └─ @radix-ui/react-dialog
|
└─ src
    └─ index.ts
```

이 방식은 단점이 꽤 많다. 패키지를 찾으려고 하면 `node_modules` 를 계속 타고 올라가며 파일을 여러 번 읽는다. `import`, `require` 속도가 느려진다. 그리고 디렉토리 크기가 너무 커진다. 실제로 파일 시스템에 디렉토리와 파일을 하나하나 만들고 쓰기 때문이다.

100개 프로젝트에서 React 18.2.0을 쓴다면 정말로 100번씩 React 18.2.0이 추가되는 것이다. 그래서 ‘[호이스팅(Hoisting)](https://toss.tech/article/node-modules-and-yarn-berry)’이라는 특이한 방법을 사용하기도 하는데, 최적화가 완전히 되는 것도 아니고 불안정하기도 해서 좋은 방법은 아니다.

#### [2] pnpm Linker

이런 단점 때문에 pnpm이 만들어짐. pnpm 문서에 'fast, disk space efficient' 한 패키지 매니저라고 써있다. 즉, 퍼포먼스가 향상된(performant) npm 이라고 함.

![image](https://github.com/user-attachments/assets/73c769bc-186f-48a7-8ac0-3d4fd7204fcd)

npm에서 문제였던 `node_modules` 를 하나씩 쓰는 것 때문에 느리고 용량도 많이 차지하는 것을 개선한 것이다.

pnpm Linker는 기존 `node_modules` 디렉토리를 그대로 사용하지만 빠르고 용량을 최적화 함. Hard link 방식 덕분이다. 보통 OS나 시스템 프로그래밍에서 파일 시스템을 관리할 때 쓰는 개념이라고 알고 있다. Hard link는 쉽게 말해 alias를 쓰는 것이다. npm 처럼 단순 복붙하는게 아니라 **alias가 생기면 거기로 바로 접근하는 것이다. 그래서 의존성이 디스크에 하나만 설치가 된다.** `node_modules` 를 쓸 때도 파일을 하나하나 쓸 필요가 없어지고, 속도도 훨씬 빠르다.

`node_modules` 파일 크기가 무척 작다. 하지만 뒤 소개할 PnP 방식 보다는 느릴 수 밖에 없다. 왜냐면 `node_modules` 디렉토리를 계속 돌면서 alias를 하나씩 걸기 때문. 그래서 약간 느리지만 npm 처럼 파일 하나씩 쓰는 건 아니기 때문에 훨씬 빠르긴 하다. 게다가 호환성도 좋다.

다만 `node_modules` 디렉토리는 그대로 유지하기 때문에, `require`, `import` 시 파일 읽기가 많이 발생, 중간중간 멈추기도 한다.

#### [3] PnP Linker

`node_modules` 디렉토리에서 벗어나고 싶다는 생각으로  래디컬하게 접근한게 PnP다. 결국 `node_modules` 없이 의존성을 처리하는 방법을 찾아냄.

PnP는 '패키지를 `import` 할 때 중요한 것은 단 두가지' 라는 관점에서 접근한다. 먼저, '어떤 파일'에서 `import` 하는가, 그리고 '무엇'을 `import` 하는가이다. 즉, 앞의 npm과 pnpm 처럼 node_modules 를 순회하는게 중요하지 않다고 생각한 것이다. 그래서 `node_modules` 디렉토리가 아니라 JavaScript 객체로 똑똑하게 처리한다.













ㄴ





































