# Server comonent vs client component

Next.js 13 이상 부터는 app route가 있다. app route를 사용하면, 모든 컴포넌트는 기본적으로 server component 형태로 사용이 되며, client component로 사용하고 싶다면 파일 상단에 `'use client'` 를 명시해주어야 함.

서버 컴포넌트는

- 서버에서 HTML로 만들어진다. 따라서, node.js api를 사용할 수 있다.
- 서버에서 그려지기 때문에 hydration이 필요가 없다.
- 상태 값도 가질 수 없다. 단순히 그리기만 하는 컴포넌트이기 때문이다.
- fetching이 가능하다. node.js에서 fetching이 가능하기 때문이다.
- 이렇게 서버에서 그려지게 되면, JavaScript 파일 코드가 필요 없어진다. -> 번들링 한 JavaScript 코드가 다이어트 된다.
- 그 전에는 페이지 단위 라우팅을 사용했을 때 페이지 단위로 렌더링 방식이 결정 되었기 때문에 서버 컴포넌트를 사용할 수 없었다.
- 따라서 모든 렌더링 방식이 있다고 하더라도 JavaScript 번들이 필요했었지만, server component를 사용한 곳은 JavaScript 번들 코드가 필요 없어진다.
- 서버 컴포넌트에서는 async, await로 만들 수도 있음. 즉, `async function Component() {}` 형태로 선언 가능. 클라이언트 컴포넌트에서는 불가능. ([참고](https://github.com/acdlite/rfcs/blob/first-class-promises/text/0000-first-class-support-for-promises.md))
  - 서버 컴포넌트는 async 함수로 정의 가능(즉, 서버 컴포넌트를 async 함수로 정의 하는 것은 전혀 이상하지 않음)
  - 다만, 서버 컴포넌트 "자체"가 아직은 실험적 기능으로 자주 보이는 패턴이 아님

클라이언트 컴포넌트는

- 서버 컴포넌트와는 다르게 클라이언트 측에서 HTML을 그려준다. (CSR)
- 상태 값을 가질 수 있다. useState, 이펙트도 가능 useEffect
- 하지만, 유저가 화면을 요청할 때 클라이언트 컴포넌트라 하더라도 초기 HTML에는 포함이 된다.
- 즉, Next.js가 초기 상태 값을 가진 상태로 초기 HTML을 그려주며 JavaScript 번들링 코드가 로드되면 hydration이 일어나, interaction이 가능해진다.
- 따라서 무조건 HTML이 그려지지 않은 형태는 아니라는 것이다.

<img width="893" alt="image" src="https://user-images.githubusercontent.com/59427983/244051212-ec72bc27-6a26-47cc-93f7-6b062afbf121.png">

<br/>

> [출처1](https://velog.io/@asdf99245/Next.js-app-router-%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C-%EC%A0%95%EB%A6%AC#server-client-%EC%96%B8%EC%A0%9C-%EC%8D%A8%EC%95%BC%ED%95%B4), [출처2](https://cat-minzzi.tistory.com/104)

## Server comopnents

![image](https://github.com/pozafly/TIL/assets/59427983/cc890658-5478-49b0-b45f-8136b9ffe9d0)

server component를 통해 전체 어플을 클라에서 렌더링 하는 것이 아닌 의도에 따라 어디에서 컴포넌트를 렌더링 할 지 정할 수 있음.

Navbar, Sidebar, Main 컴포넌트는 interactive한 부분이 없는 정적 컴포넌트이기 때문에 서버에서 server component로 렌더링할 수 있음. 나머지 interactive 한 UI는 Client component로 렌더링하게 된다. 이를 `Server-first` 접근법이라 함.

### Why Server components?

이전 클라 JavaScript 번들 사이즈에 영향을 주던 큰 의존성을 모두 서버에서 처리할 수 있게 된다. 초기 페이지 로드 속도 향상 및 클라 JS 번들 사이즈는 감소함. 기존 client 측 런타임은 캐시할 수 있고 크기를 예측 가능하고 어플이 커져도 증가하지 않는다. 추가 JS는 어플에서 사용되는 Client comonent를 통해 추가됨.

마치 페이지 단위로 SSR을 할 수 있었던 것을 컴포넌트 단위로 세분화 해 할 수 있게 된 느낌.

<br/>

## Client components

next에서 클라 컴포넌트를 먼저 서버에서 `pre-render` 된 후 클라에서 hydrate(JS가 실행되면서 intercative 하게 됨)된다.

Client component를 쓰려면 컴포넌트 파일 가장 상단에 (import 문 위) `use client` 를 선언한다.

![image](https://github.com/pozafly/TIL/assets/59427983/f33d6980-def1-4236-ac80-6c934b7edbf8)

`use client` 를 선언하면 해당 파일 안에서 의존하고 있는 다른 모듈과 child components는 client bundle에 포함된다. 기본적으로 server component가 default 이기 때문에 module 그래프의 모든 컴포넌트는 `use client` 선언문이 없다면 server component다.

<br/>

## 언제 구분하나?

- server
  - 데이터 fetching
  - 백엔드 자원에 (직접적으로) 접근
  - 민감한 정보를 서버에 유지(JWT, API Key 등등)
  - large dependencies를 서버에 유지/클라이언트 JS 번들 사이즈 감소
- client
  - interactivity, event listener(onClick 등) 필요할 때
  - state 및 라이프사이클이 필요할 때
  - browser-only API 사용
  - custom hook depend on state or browser-only API 사용

<br/>

## 패턴

### Client component는 리프노드에 둔다

client component를 사용하면 하위로 모두 client component가 되기 때문에 사용할 곳을 최소하 한다.

### Client와 Server Component 함께 쓰기

server, client 컴포넌트는 동일 컴포넌트 트리 상에서 결합될 수 있다. 리액트는 다음과 같이 렌더링한다.

- 서버에서 클라이언트에 결과를 전달하기 전 모든 Server component를 렌더링한다.
  - client 컴포넌트 안에 중첩되어 있는 server 컴포넌트 포함해 렌더링.
  - client 컴포넌트를 만나면 스킵한다.
- 클라에서 react는 클라 컴포넌트를 렌더링하고, 서버 컴포넌트의 렌더링 결과를 `slot` 에 삽입해 서버와 클라에서 수행한 작업을 병합한다.
  - 서버 컴포넌트가 클라 컴포넌트 안에 중첩된 경우, 렌더링 된 콘텐츠는 클라 컴포넌트 안에 올바르게 배치된다.

> next에서는 초기 페이지 로드 시 위 단계의 서버 컴포넌트와 클라 컴포넌트의 렌더링 결과가 모두 **서버에서 HTML로 미리 렌더링** 되어 초기 페이지 로딩 속도가 빨라짐.

### Server 컴포넌트를 client 컴포넌트안에 중첩 시키기

```jsx
'use client';
 
// This pattern will **not** work!
// You cannot import a Server Component into a Client Component.
import ExampleServerComponent from './example-server-component';
 
export default function ExampleClientComponent({
  children,
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);
 
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
 
      <ExampleServerComponent />
    </>
  );
}
```

서버 컴포넌트를 클라 컴포넌트로 import 불가능하다.

따라서 server 컴포넌트를 client 컴포넌트의 props로 전달하는 방법. 이렇게 하면 server 컴포넌트는 서버에서 렌더링 되고 client 컴포넌트가 클라에서 렌더링 되었을 때 props로 전달된 server 컴포넌트가 server 컴포넌트의 렌더링 된 결과로 표시된다.

일반적으로 `children` 을 통해 내려준다.

```jsx
'use client';

import { useState } from 'react';

export default function ExampleClientComponent({
  children,
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);

  return (
  <>
    <button onClick={() => setCount(count + 1)}>{count}</button>
    {children}
  </>
  );
}
```

이렇게 하면 `ExampleClientCompont` 는 children이 무엇인지 모른다. 자식이 결국 서버 컴포넌트의 결과에 의해 채워질 것이라는 사실조차 알지 못한다.

```jsx
// 이 패턴은 작동합니다:
// 서버 컴포넌트를 클라이언트 컴포넌트의 자식이나 소품으로 전달할 수 있습니다.
// 클라이언트 컴포넌트의 자식이나 프로퍼티로 전달할 수 있습니다.
import ExampleClientComponent from './example-client-component';
import ExampleServerComponent from './example-server-component';

// Next.js의 페이지는 기본적으로 서버 컴포넌트입니다.
export default function Page() {
  return (
  <ExampleClientComponent>
    <ExampleServerComponent />
  </ExampleClientComponent>
  );
}
```

> 정보
>
> - 이 패턴은 layout.tsx, page.tsx에 children 프로퍼티로 이미 적용되어 있어 별도의 래퍼 컴포넌트를 만들 필요가 없음.
> - JSX를 다른 컴포넌트에 전달하는 것은 새로운 개념이 아니며 항상 react 컴포지션 모델의 일부다.
> - 컴포지션 전략은 서버 컴포넌트와 클라 컴포넌트 모두에서 작동하는데, props를 받는 컴포넌트는 props가 무엇인지 알지 못하기 때문이다. 단지 전달받은 props가 어디에 배치해야하는지에 대해서만 책임이 있음.
> - 전달된 props는 클라 컴포넌트가 렌더링 되기 훨씬 전 서버에서 독립적으로 렌더링 됨.
> - 가져온 중첩된 자식 컴포넌트를 다시 렌더링하는 부모 컴포넌트의 상태 변화를 피하기 위해 '콘텐츠 리프팅'이라는 동일 전략이 사용되었음.
> - children 프로퍼티에만 국한되지 않음. 모든 프로퍼티를 사용해 JSX를 전달할 수 있음.

### Server에서 client 컴포넌트로 props 전달하기

server에서 client 컴포넌트로 전달하는 props는 `serilization` 을 해야 한다. Date, 함수, 등의 값들은 직접적으로 전달할 수 없다. 따라서, JSON.stringify로 직렬화 해야 함.
