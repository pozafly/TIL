# RSC 이해하기

> [출처](https://www.joshwcomeau.com/react/server-components/)

## SSR에 대한 빠른 입문서

서버 컴포넌트를 맥락에 맞게 배치하려면 SSR이 작동하는 방식을 알아야 함. 2015년 React 설정은 'client side' 렌더링 전략을 사용했다. 사용자는 div 태그 하나만 있는 html 을 받음

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="root"></div>
    <script src="/static/js/bundle.js"></script>
  </body>
</html>
```

스크립트 태그의 `bundle.js` 는 React 및 다른 종속성 및 개발자가 작성한 모든 코드를 포함해 어플을 마운트하고 실행하는데 필요한 모든 것이 포함되어 있음.

js가 다운로드되고 구문 분석되면 React가 실행되어 전체 어플에 대한 모든 DOM 노드를 생성하고 빈 태그에 붙인다.

이 접근 방식의 문제점은 모든 작업을 수행하는 데 시간이 걸린다는 것이다. 이 일이 진행되는 동안 사용자는 **텅 빈 흰색 화면을 보고 있다.** 새로운 기능을 개발하면 JavaScript 번들이 무거워져 사용자가 기다리는 시간을 연장시킨다.

SSR은 이런 경험을 개선하기 위해 설계 되었음. 빈 HTML 파일을 보내는 대신 서버는 실제 HTML을 생성하기 위해 어플을 렌더링한다. 사용자는 완전한 형식의 HTML 문서를 받는다.

클라에서 실행하고 상호작용을 처리하려면 여전히 React가 필요하기 때문에 `<script>` 태그는 여전히 포함되낟. 하지만 브라우저에서 React는 약간 다르게 동작하도록 구성했다. 모든 DOM 노드를 처음부터 생성하는 대신 기존 HTML을 채택했다. 이 과정을 hydration이라고 한다.

> Hydration 공급은 'dry' HTML에 인터랙티브 및 이벤트 핸들러라는 '물'을 공급하는 것과 같습니다.
>
> -> Dan Abramov

JavaScript 번들이 다운되면 react는 전체 어플을 빠르게 실행해 UI의 가상 스케치를 구축, 이를 실제 DOM에 '맞추고' 이벤트 핸들러를 연결, effect를 실행하는 등의 작업을 수행한다.

이것이 SSR이다. js 번들이 다운, 구문 분석되는 동안 사용자가 빈 흰색 페이지를 응시할 필요 없도록 서버는 초기 HTML을 생성한다. 그 후 클라측 React는 서버 측 React가 중단된 부분을 선택해 DOM을 채택하고 상호 작용을 뿌린다.

> 용어
>
> SSR의 흐름
>
> 1. myWebsite.com을 방문
> 2. Node.js 서버는 요청 수신 후 React 어플을 렌더링해 HTML을 생성함
> 3. 갓 구운 HTML이 클라로 전송됨
>
> SSR을 구현하는 한 가지 방법이지만, 유일하지는 않음. 또 다른 옵션은 어플을 **빌드** 할 때 HTML 생성함.
>
> React 어플은 컴파일되어 JSX를 일반 JavaScript로 변환하고 모든 모듈을 번들링해야 함. 동일 프로세스중 서로 다른 모든 경로에 대해 모든 HTML을 '사전 렌더링'했다면 어떻게 될까?
>
> 일반적으로 이를 SSG로 알려져 있다. SSR의 하위 변형임.
>
> 'SSR'은 여러 렌더링 전략을 포함하는 포괄적 용어임. 한가지 공통점이 있는데, 초기 렌더링은 `ReactDOMServer` API를 사용해 node.js와 같은 서버 런타임에서 발생한다는 것임. on-demand(주문형 ex. SSG) 이든, 컴파일 타임이든 실제로 이런 일이 발생하는 시기는 중요치않음.

<br/>

## 앞뒤로 튀는 중

React 데이터 페칭에 대해 알아보자. 네트워크를 통해 통신하는 두 개의 별도 어플이 있다.

- client side react app
- server side react app

React Query, SWR 또는 Apllo와 같은 것을 사용해 클라는 백엔드에 네트워크 요청 후 DB 데이터를 가져와 네트워크를 통해 다시 보낸다.

<img width="737" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/733bc43e-7d6b-4df3-a1a6-7432c369de52">

> 그래프 참고사항
>
> 여러 가지 렌더링 전략에 걸쳐 클라(브라우저)와 서버(백엔드 API) 간 데이터가 이동하는 방식을 시각화 한 것이다.
>
> 하단의 숫자는 가상의 구성 시간 단위를 나타낸다. 몇 분이나 몇 초도 아니다. **실제로 숫자는 다양한 요인에 따라 크게 달라진다.** 따라서 실제 데이터를 모델링 하는 것은 아니다.

아래는 CSR 화면이다.

<img width="695" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/36bd3eca-9b47-434d-ae79-edc7646a3e86">

사용자는 네트워크 요청이 해결되고 React가 다시 렌더링 되어 로딩 UI를 실제 콘텐츠로 대체할 때까지 이 로딩 상태를 보게 된다.

이를 설계할 수 있는 다른 방법이 있다. 다음 그래프는 동일한 일반 데이터 fetching 패턴을 유지하지만 클라 측 렌더링 대신 서버 측 렌더링을 사용한다.

<img width="708" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/2107d169-0bc9-41ea-b656-10d49d33251f">

서버에서 첫 번재 렌더링을 수행한다. 이는 사용자가 완전히 비어 있지 않은 HTML 파일을 수신한다는 의미다.

이는 개선된 사항이다. 껍데기가 빈 흰색 페이지보다 낫다. 그러나 궁극적으로 실제 중요한 방식으로 바늘을 움직이지 않는다. 사용자는 로딩 화면을 보기 위해 앱을 방문하는 것이 아니라 <u>콘텐츠</u>를 보기 위해 방문한다.

UX 차이를 실제 파악하기 위해 그래프에 몇 가지 웹 성능 지표를 추가해 보자.

<img width="723" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/7bef8701-45b4-422c-8c87-6d8f6933ca57">

<img width="721" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/488cec3d-752d-4672-b086-4729658485f3">

1. **First Paint** - 사용자는 더 이상 빈 흰색 화면을 응시하지 않는다. 일반 레이아웃이 렌더링 되었지만 콘텐츠가 여전히 누락되었다. 이를 FCP(First Contentful Paint)라고도 함.
2. **Page Interactive** - react가 다운로드 되었고 어플이 렌더링/수화 되었다. 이제 대화형 컴포넌트가 완벽하게 반응함. 이를 TTI(Time To Interactive)라고도 함.
3. Content Painted - 이제 페이지는 사용자가 관심 갖는 항목이 포함된다. DB 데이터를 가져와 UI에 렌더링 했다. 이를 LCP(Large Contentful Paint)라고도 함.

서버에서 초기 렌더링을 수행함으로써 초기 'shell'을 더 빠르게 그릴 수 있다. 진행되고 있다는 느낌을 주기 때문에 로딩 경험이 좀 더 빠르게 느껴질 수 있다. **근데 이 흐름이 좀 엉뚱한 느낌이든다.** SSR은 서버에서 요청이 시작된다는 것을 알 수 밖에 없다. 두 번째 왕복 네트워크 요청을 요구하는 대신 초기 요청 중 DB 작업을 수행하면 어떤가?

<img width="711" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/9174e8fd-cd43-411b-adde-608de334bcbd">

클라와 서버 사이를 오가는 대신 초기 요청의 일부로 DB 쿼리를 수행해 완전히 채워진 UI를 사용자에게 직접 보낸다.

이렇게 작동하려면 DB 쿼리를 수행하기 전 서버에서만 실행되는 코드 덩어리를 React에 제공할 수 있어야 함. 하지만, 이는 React에서는 옵션이 아니었다. SSR을 사용하더라도 모든 컴포넌트는 서버와 클라 모두에서 렌더링 된다.

**생태계는 이 문제에 대해 많은 해결책을 제시했다.** Next.js 및 Gatsby와 같이 서버에서만 코드를 실행하는 자체 방법을 만들었다.

예를 들어 Next.js page router는 다음과 같다.

```jsx
import db from 'imaginary-db';

// This code only runs on the server:
export async function getServerSideProps() {
  const link = db.connect('localhost', 'root', 'passw0rd');
  const data = await db.query(link, 'SELECT * FROM products');
  return {
    props: { data },
  };
}

// This code runs on the server + on the client
export default function Homepage({ data }) {
  return (
    <>
      <h1>Trending Products</h1>
      {data.map((item) => (
        <article key={item.id}>
          <h2>{item.title}</h2>
          <p>{item.description}</p>
        </article>
      ))}
    </>
  );
}
```

서버가 요청 받으면 `getServerSideProps` 함수 호출, `props` 객체를 반환함. 그리고 component로 유입되어 서버에서 먼저 렌더링 된 다음 클라에서 수화된다.

영리한 점은 `getServerSideProps` 가 클라에서 실행되지 않는 것임. JavaScript 번들에 포함되어 있지 않다.

이 방식은 시대를 훨씬 앞서갔다. 하지만, 단점이 있음.

1. 이 전략은 트리 맨 위에 있는 Component 에 대해 경로(path) 수준에서만 작동함. 다른 컴포넌트에서 이 작업을 수행할 수 없음.
2. meta framework는 고유 접근 방식을 제시함. Next.js에서는 하나의 접근 방식이 있고, Gatsby는 다른 접근 방식, Remix도 또 다른 접근이 있음. 표준이 없음.
3. 우리의 모든 React 컴포넌트는 그렇게 할 필요가 없을 때에도 항상 클라이언트에 수분을 공급함.

React 팀은 이문제를 해결하기 위해 노력함. 솔루션을 **React Server Components** 라고 함!

<br/>

## React Server Components 소개

새로운 패러다임임. <u>서버에서만</u> 실행되는 컴포넌트를 만들 수 있음. 이를 통해 component 내에서 바로 DB 쿼리 작성과 같은 작업을 수행할 수 있음.

예시

```jsx
import db from 'imaginary-db';

async function Homepage() {
  const link = db.connect('localhost', 'root', 'passw0rd');
  const data = await db.query(link, 'SELECT * FROM products');
  
  return (
    <>
      <h1>Trending Products</h1>
      {data.map((item) => (
        <article key={item.id}>
          <h2>{item.title}</h2>
          <p>{item.description}</p>
        </article>
      ))}
    </>
  );
}

export default Homepage;
```

수 년동안 react를 사용했다면 이렇게 생각할 수 있다. -> **함수 컴포넌트는 비동기실일 수 없음!** 그리고 render에서 side effect를 가질 수 없음!

**이해해야 할 핵심 사항은** 서버 컴포넌트가 다시 렌더링되지 않는다는 것이다. UI를 생성하기 위해 서버에서 `한 번` 실행된다. 렌더링 된 값은 클라로 전송되어 제자리에 고정된다. React에 관한 한 이 출력은 불변이며 결코 변경되지 않는다.

이는 React API 상당 부분이 서버 컴포넌트와 호환되지 않는다는 것이다. 예를 들어 상태는 변경될 수 있지만 서버 컴포넌트는 다시 렌더링할 수 없기 때문에 `상태(state)` 를 사용할 수 없다. 그리고 effect는 클라 렌더링 `후에` 실행되고 서버 구성요소는 클라에 전달되지 않기 때문에 effect를 사용할 수 없음.

이는 규칙과 관련해 좀 더 유연성이 있다는 것을 의미함. 예를 들어 전통적 React에서 콜백이나 이벤트 핸들러 등 내부에 side effect를 넣어 렌더링 할 `useEffect` 때마다 side effect가 반복되지 않도록 해야 함. 하지만 컴포넌트가 한 번만 실행된다면 걱정할 필요가 없음!

서버 컴포넌트 자체는 놀라울 정도로 간단하지만, 'react server component' 패러다임은 훨신 더 복잡함. 이유는 여전히 일반적인 component를 갖고 있고 서로 결합하는 방식이 혼란스러울 수 있기 때문임.

새로운 패러다임에서는 우리에게 익숙한 '전통적인' React component를 `client component` 라고 함. 컴포넌트가 클라에서만 렌더링 된다는 것을 의미하지만, 실제로는 그렇지는 않음. 클라 컴포넌트는 `클라와 서버 모두`에서 렌더링 된다!

<img width="661" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/8a1d75fe-ed81-4f01-b4cc-572bbaf86de9">

이 용어가 매우 혼란스러울 수 있지만 다음과 같이 요약할 수 있음.

- React Server Components는 새로운 패러다임의 이름임.
- 표준 React 컴포넌트가 Client Components로 이름이 변경되었음.
- Server Components는 서버에서만 렌더링된다. JavaScript 번들에 포함되지 않으므로 절대 수화되거나 다시 렌더링 되지 않음.

> `React Server Components와 SSR`
>
> 또 다른 혼란을 정리해보자. RSC는 SRR을 대체하지 않는다. RSC를 SSR 2.0으로 생각하면 안된다.
>
> 대신, 완벽하게 결합되는 두 개의 별도 퍼즐 조각, 서로를 보완하는 두 가지 맛으로 생각하자.
>
> 우리는 초기 HTML을 생성하기 위해 여전히 SSR 의존하고 있다. RSC는 그 위에 구축되어 client 측 JavaScript 번들에서 특정 component를 생략해 서버에서만 실행되도록 할 수 있다.
>
> 실제로 SSR 없이 RSC를 사용하는 것도 가능하지만, 실제로는 함께 사용하면 더 나은 결과를 얻을 수 있다. 예시 React 팀이 SSR 없이 [최소한의 RSC 데모를 구축했다.](https://github.com/reactjs/server-components-demo)

<br/>

## 호환 가능 환경

React 최신 버전을 설치한다고 RSC를 바로 사용할 수 없다. 유일한 방법은, Next.js 13.4+에서 App Router를 사용하는 것이다.

<br/>

## Client Components 지정

```jsx
'use client'; // here

import React from 'react';

function Counter() {
  const [count, setCount] = React.useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Current value: {count}
    </button>
  );
}

export default Counter;
```

`'use client'` 지시어를 사용하면 클라에서 다시 렌더링할 수 있도록 JS 번들에 포함되어야 함을 React에 알리는 방법이다. `'use server'` 는 지정하지 않는다.

> 어떤 component가 client component가 되어야 하나?
>
> 일반적으로 컴포넌트가 server 컴포넌트가 될 수 있는 경우 server 컴포넌트 여야 한다. server 컴포넌트는 추론하기 더 간단하고 쉽다. 성능상으로 이점도 있다. 서버 컴포넌트는 클라이언트에서 실행되지 않기 때문에 해당 코드가 JS 번들에 포함 안된다. 서버 컴포넌트는, TTI(Page Interactive) 측정항목을 향상시킬 수 있는 잠재력이 있다.
>
> 그렇지만, **가능한 많은 클라 컴포넌트를 근절하는 것을 목표로 삼아서는 안된다**! 최소한의 클라 컴포넌트 수에 맞춰 최적화하려고 해서는 안된다. 지금까지 모든 React 앱의 모든 React 컴포넌트는 클라 컴포넌트였다는 점을 기억할 가치가 있다.
>
> 서버 컴포넌트로 작업하면 매우 직관적이라는 것을 알게 될 것이다. 일부 컴포넌트는 state 변수나 effect를 사용하기 때문에 클라이언트에서 실행해야 한다. 이런 컴포넌트에는 `'use client'` 지시어를 넣자.

<br/>

## 경계

RSC에 익숙해졌을 때 `props` 가 변경되면 어떻게 되나?

```jsx
function HitCounter({ hits }) {
  return (
    <div>
      Number of hits: {hits}
    </div>
  );
}
```

초기 server side render에서 `hits` 가 `0` 과 같다고 해보자. HTML을 이렇게 생성한다.

```html
<div>
  Number of hits: 0
</div>
```

하지만, `hits` 가 변하면 어떻게 될까? 0 -> 1로 변경되었다 해보자. `HitCounter` 는 다시 렌더링 되어야 하지만, server components이기 때문에 다시 렌더링 할 수 없다!

**문제는 서버 컴포넌트를 단독으로 분리하면 실제로 의미가 없다는 것이다.** 우리는 어플 구조를 고려하기 위해 좀 더 전체적 관점을 취하기 위해 축소해야 한다.

<img width="584" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/fa321b8c-ba61-4848-9e86-232d6d609325">

이런 컴포넌트가 모두 서버 컴포넌트라면 의미가 있음. 어떤 컴포넌트도 다시 렌더링 되지 않으므로 props 중 어떤 것도 변경되지 않는다.

`Article` 컴포넌트가 클라 컴포넌트라 해보자. 상태를 사용하려면 이를 클라 구성요소로 변환해야 한다.

<img width="573" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/cde1a224-71c4-4989-bcac-2d1117f10996">

`Article` 컴포넌트가 리렌더링 될 때 하위 컴포넌트 모두 다시 리렌더링 된다. 그러나 서버 컴포넌트는 다시 렌더링 할 수 없음.

이런 불가능한 상황을 방지하기 위해 규칙이 있음. `클라 컴포넌트는 다른 클라 컴포넌트만 가져올 수 있다.` 'use client' 지시어는 이런 인스턴스가 클라이언트 구성 요소가 되어야 함을 의미한다.

<img width="553" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/41a45774-07af-40b7-a44e-5d1a3afba7fd">

컴포넌트에 `'use client'` 지시어를 포함하면 `Article` 에 **컴포넌트 경계**가 생성된다. 이 경계 내 모든 컴포넌트는 암시적으로 클라이언트 컴포넌트가 된다. `HitCounter` 컴포넌트에 'use client' 지시어가 없더라도 클라이언트에 수화/렌더링 된다.

따라서, `'use client'` 는 클라에서 실행해야 하는 모든 단일 파일에 추가할 필요가 없고, 새 클라이언트 경계를 만들 때만 추가하면 된다.

<br/>

## 해결 방법

클라 컴포넌트는 서버 컴포넌트를 렌더링할 수 없다는 사실을 알았을 때, 제한적이라고 느낀다. 어플에서 상태를 끌어올려 사용해야 하는 경우는 어떻게 해야하나? 모든 것이 클라 컴포넌트가 되어야 한다는 뜻인가?

소유자가 변경 되도록 어플을 재구성 해 이 제한을 해결할 수 있는 것으로 나타났음.

```jsx
'use client';

import { DARK_COLORS, LIGHT_COLORS } from '@/constants.js';

import Header from './Header';
import MainContent from './MainContent';

function Homepage() {
  const [colorTheme, setColorTheme] = React.useState('light');

  const colorVariables = colorTheme === 'light'
    ? LIGHT_COLORS
    : DARK_COLORS;

  return (
    <body style={colorVariables}>
      <Header />
      <MainContent />
    </body>
  );
}
```

다크 모드와 라이트 모드 전환하도록 state를 사용해야 함. 어플 트리 상 `<body>` 에서 실행해야 함.

이 문제를 해결하려면 색상 관리 항목을 자체 컴포넌트에 집어 넣고 자체 파일로 이동해보자.

```jsx
// /components/ColorProvider.js
'use client';

import { DARK_COLORS, LIGHT_COLORS } from '@/constants.js';

function ColorProvider({ children }) {
  const [colorTheme, setColorTheme] = React.useState('light');

  const colorVariables = colorTheme === 'light'
    ? LIGHT_COLORS
    : DARK_COLORS;

  return (
    <body style={colorVariables}>
      {children}
    </body>
  );
}
```

`Homepage` 에 입혀보자.

```tsx
// /components/Homepage.js

import Header from './Header';
import MainContent from './MainContent';
import ColorProvider from './ColorProvider';

function Homepage() {
  return (
    <ColorProvider>
      <Header />
      <MainContent />
    </ColorProvider>
  );
}
```

더 이상 상태를 사용하지 않기 때문에 `'use client'` 를 사용하지 않아도 된다. `Header`, `MainContent` 컴포넌트는 더 이상 클라이언트 컴포넌트로 암시적으로 변환되지 않는다는 뜻이다.

하지만, `ColorProvider` 컴포넌트는 클라이언트 컴포넌트고, `Header`, `MainContent` 컴포넌트는 하위다. 하지만, 클라이언트 경계에 있어서는 부모/자식 관계가 중요하지 않음. Header와 MainContent를 가져오고 렌더링 하는 것은 `HomePage` 임. 즉, 컴포넌트의 props를 결정하는 것은 HomePage임.

우리가 해결하려는 문제는 server 컴포넌트가 리렌더링할 수 없으므로 해당 props에 새로운 값을 부여할 수 없다는 점이다. 이 새로운 설정에서는 홈페이지가 Header, MainContent 컴포넌트에 대한 props를 결정하고, HomePage가 서버 컴포넌트이므로 문제가 없다.

이 문제는 머리 아프다.

더 정확히 말하면 `'use client'` 는 파일/모듈 수준에서 작동한다. 클라 컴포넌트 파일에서 가져온 모든 모듈도 클라 컴포넌트여야 한다. 번들러가 코드를 번들로 묶을 때 결국 이런 임포트된 모듈을 따르게 된다!

> 테마를 변경하면 어떻게 되나?
>
> 예제에서는 변경할 수 있는 방법이 없다. `setColorTheme` 함수를 사용하는 곳이 없다. 생략된 것이다. 전체 예제 에서는 Context를 사용해 모든 자손이 setter 함수를 사용할 수 있다. context를 소비하는 컴포넌트가 클라 컴포넌트인 한 모든것이 잘 작동한다.

<br/>

## Hood 아래 엿보기

낮은 수준에서 보자. 서버 컴포넌트는 어떤 모습인가? 실제로 무엇이 생성되나?

```jsx
function Homepage() {
  return (
    <p>
      Hello world!
    </p>
  );
}
```

서버 컴포넌트다.

```html
<!DOCTYPE html>
<html>
  <body>
    <p>Hello world!</p>

    <script src="/static/js/bundle.js"></script>
    <script>
      self.__next['$Homepage-1'] = {
        type: 'p',
        props: null,
        children: "Hello world!",
      };
    </script>
  </body>
</html>
```

브라우저에서는 이런 모습을 갖고 있다.

> html 코드는 약간의 생략 코드다.
>
> RSC 컨텍스트에서 생성된 실제 JS는 html 문서 파일 크기를 줄이기 위한 최적화로 문자열화 된 JSON 배열을 사용한다.

HTML 문서에 react 앱에서 생성된 UI인, `Hello world!` 단락이 포함되어 있다. 이는 SSR 때문이며, React 서버 컴포넌트에 직접적으로 기인한 것은 아니다.

그 아래에 JS 번들 로드하는 스크립트 태그가 있다. 이 번들에는 어플에서 사용되는 클라 컴포넌트 뿐 아니라 react와 같은 의존성이 포함되어 있다. 그리고 홈페이지 컴포넌트는 서버 컴포넌트이기 때문에 해당 컴포넌트의 코드는 이 번들에 포함되지 않는다.

마지막으로 인라인 JS 가 포함된 두 번째 `<script>` 태그가 있다.

```javascript
self.__next['$Homepage-1'] = {
  type: 'p',
  props: null,
  children: "Hello world!",
};
```

흥미로운 부분이다. React에게 '`HomePage` 컴포넌트 코드가 누락되었단 것을 알지만 걱정 마라. 렌더링 된 내용은 다음과 같다' 라고 말하는 것이다.

일반적으로 React는 클라에서 수화할 때 모든 컴포넌트를 빠르게 렌더링해 어플의 가상 표현을 구축한다. 서버 컴포넌트의 경우 코드가 JS 번들에 포함되어 있지 않기 때문에 그렇게 할 수 없다.

따라서 서버에서 생성된 가상 표현인 렌더링 된 값을 함께 전송한다. React는 클라에서 로드될 때 설명을 다시 생성하는 대신 해당 설명을 재사용한다.

이것이 `ColorProvider` 위의 예가 작동하는 이유다. `Header`, `MainContent` 의 출력은 자식 props를 통해 `ColorProvider` 로 전달된다. `ColorProvider` 는 원하는 만큼 리렌더링할 수 있지만, 이 데이터는 서버에 의해 고정된 정적 데이터다.

서버 컴포넌트가 어떻게 직렬화 되어 네트워크를 통해 전송되는지 실제로 보고 싶다면 [RSC Devtools](https://www.alvar.dev/blog/creating-devtools-for-react-server-components) 를 봐라.

> **서버 컴포넌트는 서버가 필요 없다.**
>
> SSR은 렌더링 전략을 포괄하는 다양한 전략이라 말했다.
>
> - static: 어플이 빌드 시 HTML이 생성됨
> - dynamic: 사용자가 페이지 요청시 '온디맨드'로 html이 생성됨.
>
> React Server 컴포넌트는 이런 렌더링 전략 중 어떤 것과도 호환된다.
>
> 즉, 서버 없이도 RSC를 사용할 수 있다. 정적 HTML 파일을 여러 개 생성해 원한느 곳에 호스팅할 수 있다. 실제로 이것이 Next.js 앱 라우터에서 기본적으로 일어나는 일이다. '온디멘드'로 작업을 수행해야 하는 경우가 아니라면, 이 모든 일은 빌드 중 미리 수행된다.

> React가 전혀 없나?
>
> 어플에 클라 컴포넌트를 포함하지 않는다면 실제 React를 다운 해야 할까? React 서버 컴포넌트를 사용해 정말 정적 no-JS 웹사이트를 구축할 수 있을까?
>
> 문제는 RSC는 Next.js 프레임워크 위에서만 사용할 수 있으며, 이 프레임워크에는 라우팅과 같은 것을 관리하기 위해 클라에서 실행해야 하는 코드가 많이 포함되어 있다.
>
> 하지만, 직관적으로 생각하면 오히려 더 나은 사용자 경험을 제공하는 경향이 있다. 예를 들어 Next의 라우터는 완전히 새로운 HTML 문서를 로드할 필요가 없으므로 일반적 a 태그보다 링크 클릭을 더 빠르게 처리한다.
>
> 잘 구조화 된 Next.js 어플은 JS가 다운로드 되는 동안에도 계속 작동하지만, JS가 로드되면 훨씬 더 빠르고, 개선된다.

<br/>

## 장점

<img width="734" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/87821c25-de8f-464d-a1ab-0fc7aadee07b">

<img width="733" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/fef99dd1-9827-4eef-82c5-b2f41daa5ed6">

Download JavaScript, Hydration 부분이 after에서 더 줄어듦.

하지만, 이것이 가장 흥미롭지 않은 부분임. 솔직히 대부분의 Next.js 앱은 '페이지 인터렉티브' 타이밍에 있어 충분히 빠르다.

시맨틱 HTML 원칙을 따른다면, 대부분의 앱은 React가 수화되기 전에도 작동할 것이다. 링크를 따라갈 수 있고, 양식을 제출할 수 있으며 아코디언을 확장하고 축소할 수 있다. (`<details>`, `<summary>` 사용). 대부분 플젝에서 react가 하이드레이트 되는 데 몇 초가 걸리더라도 괜찮다.

하지만 정말 멋진 점은, 더 이상 번들 크기와 기능 면에서 타협할 필요가 없다는 점이다!

예를 들어 대부분 기술 블로그에는 일종의 구문 강조 표시 라이브러리가 필요하다. 하지만 구문 강조 라이브러리는 수 메가 바이트로 JS 번들에 넣기에는 너무 크다. 따라서 업무에 필수적이지 않은 언어와 기능을 제거해 타협을 해야함.

이것이 바로 RSC와 함께 작동하도록 설계된 최신 구문 강조 패키지인 Bright의 핵심 아이디어다.

![image](https://github.com/pozafly/TIL/assets/59427983/18683b01-0538-41e8-87fa-48b3449ed171)

이게 좋은 점이다. JS 번들에 넣기에 너무 큰 것이다. JS 번들에 포함하기에 비용이 너무 많이 들기 때문에 서버에서 무료로 실행할 수 있고, 번들어 0kB를 추가하고 더 나은 UX를 제공하는 것.

성능과 UX 뿐만이 아니다. RSC를 사용하면서 얼마나 간편한지 알게 되었음. 종속성 배열, 오랜된 클로저, 메모화 또는 변경사항으로 인해 발생하는 기타 복잡한 문제에 대해 걱정할 필요가 없다.

<br/>

## 전체 그림

RSC는 모던 리액트라는 퍼즐의 한 부분이다.

RSC와 suspense 및 새로운 스트리밍 SSR 아키텍쳐를 결합하면 상황이 정말 흥미로워 짐. 아래와 같은 것을 할 수 있음.

<img width="715" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/a65b0b63-8c39-42c2-b401-229e0a9b0cdd">

[Github](https://github.com/reactwg/react-18/discussions/37) 에서 확인할 수 있음.
