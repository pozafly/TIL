# Fragments

## react에서 fragments가 나온 이유

1. 여러 요소 반환: React 컴포넌트는 기본적으로 단일 루트 요소만 반환할 수 있다. 예를 들어, 컴포넌트 내에서 여러 개의 `<div>` 요소를 반환하려고 하면 오류가 발생한다. 이런 경우 Fragments를 사용하여 여러 요소를 묶을 수 있다. Fragments를 사용하면 불필요한 추가 DOM 노드를 생성하지 않고 컴포넌트의 구조를 유지할 수 있다.
2. 추가 DOM 노드 방지: 이전에는 컴포넌트 내에서 여러 요소를 반환하기 위해 `<div>` 등의 불필요한 래퍼 요소를 사용해야 했다. 그러나 이는 DOM 트리에 추가 노드를 생성하므로 불필요한 렌더링 비용이 발생할 수 있다. Fragments를 사용하면 추가 DOM 노드를 방지하면서 컴포넌트 구조를 유지할 수 있다.
3. 스타일링 충돌 방지: 컴포넌트를 스타일링할 때 외부 라이브러리나 스타일 가이드에 따라 특정 요소를 래핑해야 할 수도 있다. Fragments를 사용하면 스타일링 요구사항을 충족시키면서도 컴포넌트 구조를 깨지 않을 수 있다.

<br/>

## 왜 react에서  컴포넌트는 기본적으로 단일 루트 요소만 반환할 수 있나?

React에서 컴포넌트는 기본적으로 단일 루트 요소만 반환할 수 있는 이유는 `JSX(JavaScript XML)`의 작동 방식과 가독성, 유지 보수 측면을 고려한 설계 결정이다.

1. JSX 구문의 가독성: JSX는 JavaScript와 XML을 결합한 문법으로, 가독성이 뛰어나고 컴포넌트의 구조를 쉽게 이해할 수 있도록 돕는다. 단일 루트 요소를 반환함으로써 컴포넌트의 구조를 명확히 만들 수 있고, 컴포넌트의 구조를 파악하기 쉬워진다.
2. 컴포넌트의 재사용성: 단일 루트 요소를 반환하면 해당 컴포넌트를 더 쉽게 재사용할 수 있다. 단일 루트 요소를 가정하면, 해당 컴포넌트를 다른 컴포넌트 내부에 쉽게 삽입하거나 조합할 수 있다. 또한, 컴포넌트를 추상화하고 별도의 단일 루트 컴포넌트로 래핑함으로써 재사용성을 높일 수 있다.
3. React 엘리먼트의 추적과 관리: React는 가상 DOM을 사용하여 컴포넌트의 변경 사항을 추적하고 업데이트하는데, 이를 위해 컴포넌트의 루트 요소를 식별해야 한다. 단일 루트 요소를 반환하면 React는 해당 요소를 식별하여 변경 사항을 효율적으로 감지하고 업데이트한다. 이를 통해 성능을 최적화하고 예상치 못한 동작을 방지할 수 있다.

물론, 단일 루트 요소를 반환해야 하는 경우가 아니라면 Fragments를 사용하여 여러 요소를 그룹화하고 반환할 수 있다. Fragments를 사용하면 여러 요소를 감싸는 불필요한 래퍼 요소를 생성하지 않고도 JSX를 작성할 수 있다.