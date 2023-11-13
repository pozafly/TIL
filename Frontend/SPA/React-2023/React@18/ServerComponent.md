# ServerComponent

> [출처](https://ykss.netlify.app/translation/everything_i_wish_i_knew_before_moving_50000_lines_of_code_to_react_server_components/)

오래전 PHP와 같은 기술을 사용해 서버에서 웹사이트를 생성했다. 이는 비밀번호를 이용해 데이터를 가져오고, 클라이언트가 가벼운 HTML 페이지를 손쉽게 받을 수 있도록 대형 컴퓨터에서 CPU 집약적 작업을 처리하는데 유용했다.

![image](https://github.com/pozafly/TIL/assets/59427983/a08eaa4c-3c0b-477f-9fd0-d39ef1ebfb31)

더 빠른 응답과 상호작용하려면? 사용자가 어떤 행동을 취할 때마다 쿠키를 서버로 다시 전송하고 서버가 완전히 새로운 페이지를 생성하도록 해야 하나? 대신 클라이언트가 그 작업을 수행하도록 모든 렌더링 코드를 JavaScript 클라이언트에 전송하면 된다.

이를 CSR(Client Side Rendering) 또는 SPA(Single Page Application)이라 하는데, [이전엔 나쁜 방법으로 생각](https://begin.com/blog/posts/2023-02-21-why-does-everyone-suddenly-hate-single-page-apps)했음. 

대시보드와 같이 자주 바뀌고 상호작용이 많은 페이지의 경우 SPA가 적합함. 하지만 검색 엔진이 페이지를 읽도록 하고 싶은데 검색엔진이 JavaScript를 읽지 않으면? 서버에서 보안을 유지해야 하는 경우라면? 사용자 디바이스의 전력이 낮거나 연결 상태가 좋지 않은 경우라면?

![image](https://github.com/pozafly/TIL/assets/59427983/005a4092-a5b6-4146-a929-cb6ffa7b19e5)

SSR과 SSG가 바로 이 지점에 등장함. Next.js, Gatsby 같은 도구는 SSR, SSG를 사용해 서버에서 페이지를 생성 후 HTML과 JavaScript를 클라이언트로 전송한다. 두 가지 장점 모두 누릴 수 있었다. 클라에서 HTML을 즉시 표시해 사용자에게 볼거리를 제공한다. 그리고 JavaScript 가 로드되면 사이트가 인터렉티브해진다. 보너스로 검색엔진이 HTML을 읽을 수 있다.

![image](https://github.com/pozafly/TIL/assets/59427983/83f5ade5-99db-4db7-8837-eaf9cc19b5f4)

하지만 문제가 있음. SSR/SSG 접근 방식은 페이지를 생성하는데 사용된 JavaScript를 클라이언트로 전송하고, 클라이언트는 이 모든 것을 다시 실행하여 로드된 JavaScript 와 HTML을 결합한다.(참고로 이 결합을 하이드레이션이라 함) 그 모든 자바스크립트를 전송하고 실행해야 하나? 하이드레이션을 위해 모든 렌더링 작업을 굳이 중복해야 하나?

두 번째로 SSR이 오래 걸리면 어쩌나? 많은 코드를 실행하거나 느린 DB호출을 기다리는 중일 수 있다. 그러면 사용자는 계속 기다리며 짜증 남. 이때 서버 컴포넌트가 등장한다.

<br/>

## React 서버 컴포넌트란?

RSC는 클라가 아닌 서버에서 실행되는 React 컴포넌트다. 하지만, '무엇' 보다 '왜'가 훨씬 더 흥미롭다. SSR에 비해 두 가지 큰 장점이 있다.

첫째, RSC를 지원하는 프레임워크는 코드가 실행되는 위치를 정의할 수 있는 방법을 제공한다. PHP 시절처럼 서버에서만 실행해야 하는 것과 클라에서 실행해야 하는 것 (SSR 처럼)을 정의할 수 있다. 각각 서버 컴포넌트와 클라이언트 컴포넌트라 함. 코드가 실행되는 위치를 명시할 수 있으므로 클라에 보내는 JavaScript 양을 줄일 수 있어 번들 크기가 작아지고 **하이드레이션 과정에서 작업량을 줄일 수 있음**.

![image](https://github.com/pozafly/TIL/assets/59427983/676ab0d3-7ddc-49a2-ab27-3bb806b9cb36)

두 번째 장점은, **서버 컴포넌트의 경우 컴포넌트 내에서 직접 데이터를 가져올 수 있다는 점**. 데이터 가져오기가 완료되면 서버 컴포넌트는 해당 데이터를 클라이언트로 *스트리밍* 할 수 있다.

새로운 데이터 페칭 방식은 두 가지 측면에서 변화를 가져온다. 첫 번째로 이제 React에서 데이터를 불러오는 것이 훨씬 더 쉬워졌다. 모든 서버 컴포넌트는 노드 라이브러리를 사용하거나 우리가 모두 잘 알고 있는 `fetch` 함수를 사용해 직접 데이터를 불러올 수 있다. 사용자 컴포넌트는 사용자 데이터를, 동영상 컴포넌트는 동영상 데이터를 가져올 수 있다. 더 이상 라이브러리를 사용해 복잡한 로딩 상태를 관리(react-query는 여전히 좋음) 하기 위해 `useEffect` 를 사용할 필요가 없으며, 페이지 수준에서 `getServerSideProps` 로 많은 데이터를 가져온 다음 필요한 컴포넌트로 일일이 내려보내지 않아도 된다.

두 번째로, 앞서 언급한 문제를 해결한다. DB 호출이 느린가? 응답을 기다릴 필요 없이 느린 컴포넌트가 준비되면 클라에 전송하기만 하면 된다. 사용자는 그 동안 사이트의 나머지 부분을 즐길 수 있다.

![453e68deb5faa8fc744571fde66048773c8d65ca-1456x880](https://github.com/pozafly/TIL/assets/59427983/6b2c017b-c49e-40df-9fe7-12c4d0678581)

그럼, 폼 제출과 같은 사용자의 클라 작업에 대한 응답으로 서버에서 데이터를 가져와야 한다면? 클라가 서버로 데이터를 전송하면 서버는 데이터를 가져오는 등 작업 수행 후 스트리밍 한 것처럼 응답을 클라이언트로 다시 스트리밍할 수 있다. 이 양방향 통신은 엄밀히 말해 React 서버 컴포넌트가 아니라 [React Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/forms-and-mutations#actions)이지만, 같은 기반 위에 구축되어 있고 밀접하게 연관되어 있다.

<br/>

## React 서버 컴포넌트 단점

RSC가 CSR과 SSR 보다 훨씬 낫나? 하지만 문제가 있다. 다음은 마이그레이션 할 때 가장 많은 시간을 할애했던 세 가지 부분이다.

### 현재 CSS in JS를 사용할 수 없다

현재로서는 [서버 컴포넌트에서 CSS-in-JS가 작동하지 않는 것](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration#step-7-styling)으로 밝혀짐. (하지만, styled-components 가능, emotion은 지원 예정 중) [styled-components](https://styled-components.com/)에서 [Tailwind CSS](https://tailwindcss.com/)로 전환하는 것이 RSC 전환의 가장 큰 부분이었지만, [수고할 만한 가치가 있었다고 생각.](https://www.mux.com/blog/the-building-blocks-of-great-docs#tailwind-css)

CSS-in-JS에 올인했다면 할 일이 많을 수 있다.

### 서버 컴포넌트에서 React Context가 작동하지 않는다

[React Context](https://react.dev/learn/passing-data-deeply-with-context)는 [클라이언트 컴포넌트*에서만*](https://nextjs.org/docs/app/building-your-application/rendering#rendering-third-party-context-providers-in-server-components) 접근할 수 있다. 서버 컴포넌트 간에 props를 사용하지 않고 데이터를 공유하려면 아마도 일반적인 [모듈](https://nextjs.org/docs/app/building-your-application/rendering#sharing-data-between-server-components)을 사용해야 할 것이다.

중요한 핵심이 있다. 어떤 중류의 데이터를 React 어플리케이션 하위 트리로 제한하고 싶다면, 서버 컴포넌트에는 이를 위한 훌륭한 메커니즘이 없다. React Context를 많이 사용할 곳도 상호작용이 많았고, 클라에 제공해야 하는 부분이었기 때문이다. 예를 들어 검색 경험은 컴포넌트 트리 전체에 걸쳐 `queryString` 및 `isOpen`과 같은 상태를 공유한다.

### 모든 것을 한번에 머릿속에 담아두는게 어렵다

RSC는 코드가 실행되는 위치와 데이터 가져오기 방식에 대해 더 많은 유연성을 제공한다. **유연성에는 복잡성이 수반된다**. 어떤 두구도 이 [복잡성을 완전히 덮을 수](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)는 없으므로 언젠가는 개발자가 이 복잡성을 이해하고 직면하여 다른 개발자와 소통해야 함.

새로운 개발자가 코드 베이스를 처음 접할 때마다 이런 질문이 제기 됨. '서버에서 무엇이 실행되고 있나? 클라에서는 무엇이 실행되고 있나?' 그리고 [캐싱](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating)의 복잡성은 말할 것도 없다.

React 서버 컴포넌트를 어떻게 사용할지, 점진적 마이그레이션은 어떻게 할지, 읽기 어려운 스파게티 코드 덩어리를 만들지 않고, 까다로운 작업을 수행햐려면 어떻게 해야 할지에 대해 이야기해보자.

---

