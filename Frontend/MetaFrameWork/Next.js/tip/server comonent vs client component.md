# server comonent vs client component

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