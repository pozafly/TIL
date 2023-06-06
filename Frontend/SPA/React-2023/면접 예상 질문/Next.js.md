# Next.js

- Hydration이란?
  - 서버로 부터 전달받은 HTML과 React component를 일치시키는 기법.
  - Hydrantion이 완료되면, 사용자와 어플리케이션이 상호작용이 가능해진다. JavaScript 코드와 HTML이 연결되는 과정을 hydration이라고 한다.
  - 예를 들면, 버튼에 이벤트 핸들러 추가 등이 있다.
  - Hydration 시 서버에서 한번 렌더링하고, 클라이언트에서 한번 렌더링 하면 비효율적인 것이 아닌가?
    - 아니다. 서버에서 pre-rendering 후, 유저에게 빠른 웹 페이지로 응답할 수 있는 장점이 있다. 실제 paint 되는 과정이 실행되지는 않기 때문에 비효율적이지는 않다.
  - Hydration은 Next.js에서만 일어나나?
    - 아니다. ReactDOM.hydration() 함수를 통해 React에서도 실행시킬 수 있다.
  - ReactDOM.hydration() 함수는 어떤 일은 하나?
    - 서버에서 받아온 DOM tree와 차제적으로 렌더링 한 tree를 비교한다.
    - 두 tree 사이의 diff를 얻어내고, 자체적으로 (클라이언트 사이드) 렌더링 한 tree에 비교하면서 어떤 DOM이 어떻게 매칭되는지 이해한다.
    - 이해한 내용에 따라 CSR 동작을 실행한다.
- 렌더링 방식 App Route, Page Route 무슨 차이인가?
  - Page Route
    - 페이지 단위로 렌더링 방식을 규정한다.
    - getStaticProps를 사용하면 SSG, getServerSideProps를 사용하면 SSR.
  - App Route
    - 서버 컴포넌트, 클라이언트 컴포넌트가 나오면서 컴포넌트 단위로 렌더링 방식을 규정한다.

