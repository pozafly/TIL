# Suspense SRR Architecture in React 18

> [출처](https://blog.mathpresso.com/suspense-ssr-architecture-in-react-18-ec75e80eb68d)

## What is SSR?

어플을 로드했을 때, 가능한 빠르게 fully-interactive 한 페이지를 보여주고 싶을 것이다.

아래 사진의 초록색 영역은 상호 작용이 가능하다(interactive)는 것을 나타낸다. 해당 영역에 JavaScript 이벤트 핸들러가 붙어 있는 상태고, 버튼을 누르면 상태를 업뎃 하는 등 Reactive 한 동작들을 수행할 수 있다는 것을 의미함.

![image](https://github.com/pozafly/TIL/assets/59427983/f9545eeb-d685-46de-b412-727e30b5aea0)

이 상태가 되려면 JavaScript가 **모두** 로드 되어야 함. 어플이 작다면 로딩 시간도 작을 것임. 만약 SSR을 사용하지 않는다면 유저는 로딩 때 빈화면을 볼 것임.UX가 좋지 않다. 특히 JavaScript 로딩 시간 오래걸리는 저사양 디바이스에서 그렇다. 그런 이유로 SSR이 권장된다.

SSR은 component를 서버에서 HTML로 렌더링 해 유저에게 내려주는 방식으로 동작함. HTML은 브라우저에서 기본적으로 제공하는 link, form input 과 같은 상호작용 요소를 제외하고는 interactive 하지는 않지만, 유저들로 하여금 JavaScript가 로딩되는 동안 무언가 볼 수 있게 해준다.

아래 사진에서 회색 영역은 상호작용을 할 수 없다는 의미다. JavaScript가 아직 로드되지 않아 버튼이 있지만 눌렀을 때 state가 업뎃이 되지 않음. 하지만, 콘텐츠가 많은 경우 콘텐츠를 읽을 수 있으므로 SSR을 사용하는게 유용하다.

![image](https://github.com/pozafly/TIL/assets/59427983/313e48e0-45d4-4e03-b3c3-fc66a25c2976)

React와 어플이 모두 로드되면 회색 영역의 HTML이 상호작용하도록 만들어야 함.

리액트는 메모리상에 컴포넌트를 렌더링 하지만, DOM 노드를 생성하지 않고(이미 노드가 문서에 존재하므로) 이미 존재하는 HTML에 이벤트 핸들러와 같은 로직을 붙여줌. 이렇게 React가 로드되고 나서 컴포넌트를 메모리에 렌더링하고 이벤트 핸들러를 붙이는 일련의 과정을 Hydration 이라고 함.

Hydration이 끝나면 '평소의 리액트'(React as uaual)다. 컴포넌트는 상태 값을 변경할 수도 있고, 클릭에 반응할 수도 있다.

![image](https://github.com/pozafly/TIL/assets/59427983/f9545eeb-d685-46de-b412-727e30b5aea0)

SSR은 마법처럼 보일 수 있지만 이 과정 자체로는 상호작용을 더 빠르게 만들지 않는다. 대신, 상호작용이 없는 부분을 더 빠르게 볼 수 있게 해 유저들이 JavaScript가 로드되는 동안 정적 콘텐츠를 볼 수 있도록 해준다. SSR은 특히 네트워크 상태가 좋지 않은 유저들에게 큰 차이를 가져옴. 쉬운 인덱싱과 빠른 속도로 SEO에 도움을 준다.

<br/>

## 오늘날 SSR의 문제점은 무엇인가?

SSR은 많은 장점이 있지만, 근본적 문제를 해결하지 못함.

### 모든 것을 가져와야 표시할 수 있다

컴포넌트가 '데이터를 기다리도록' 하지 않는다는 것이다. 현재 SSR 관련 API를 사용하면 HTML을 렌더하는 시점에 서버에서 컴포넌트 렌더를 위해 필요한 모든 데이터가 준비되어야 하고, 이 방법은 매우 비효율적임.

예를 들어 댓글이 있는 포스트를 렌더링 하고 싶다 가정해보자.
