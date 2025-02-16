# Hooks

## Hooks

- Hooks란?
  - 클래스 컴포넌트를 사용할 필요 없이 상태 값과 여러 React의 기능을 사용할 수 있도록 만든 것.
  - 함수 컴포넌트에서 state와 생명주기 기능을 연동할 수 있게 해주는 함수다.
  - 장점은?
    - 재사용성이 좋다. 고차 컴포넌트로 재사용성을 이용했지만, 고차 컴포넌트 위계가 많아질 수록 wrapper hell이 발생했다. 그리고 코드의 추적이 어려웠다.
    - hook은 계층의 변화 없이 상태 관련 로직을 재사용할 수 있도록 도와준다.

## useState

- useState란?
  - 컴포넌트에 상태 변수를 추가할 수 있게 해주는 Hook.
- 업데이터 함수는 무엇인가?
  - state를 인수로 사용하고 다음 state를 반환하는 함수다.
  - state는 스냅샷으로 처리되기 때문에 setState를 연속으로 사용할 경우 원하는 결과를 얻지 못할 때 사용한다.
  - 이벤트 핸들러에서 처리된 로직 이후 업데이터 함수가 큐에 들어가고, 큐에서 순차적으로 순회하며 함수가 실행 된 후 state를 업데이트 한다.
- state가 스냅샷으로 처리되는 이유와 업데이터 함수가 존재하는 이유?
  - state 업데이트를 하기 전 작업이 큐에 들어가고, 큐에서 처리가 끝난 후 리렌더링을 하는데, setState가 일어날 때마다 리렌더링이 일어나면 효율적이지 않기 때문이다.
  - 따라서 Batching(일괄처리) 후에 리렌더링이 일어난다. 업데이터 함수는 이 작업에서 꼬일 수 있는 문제를 해결한다.
- Batching 이전에 DOM을 조작하기 위해서는 어떤 것을 사용할 수 있나?
  - `flushSync`를 사용할 수 있음.
  - `flushSync` 란?
    - 콜백 함수 내부의 업데이트를 동기적으로 하도록 하는 함수. 사용하면 DOM이 즉시 업데이트 된다. 즉, 먼저 setState를 해서 화면을 업데이트 한 후, 그 다음 로직을 실행할 수 있다.
    - 성능을 크게 저하시킬 수 있기 때문에 아껴서 사용하라.
    - 언제 사용하나?
      - 브라우저 API나 UI 라이브러리와 같은 서드파티 코드와 통합할 때 사용한다.

---

## useEffect

- useEffect란?
  - 컴포넌트를 외부 시스템과 동기화할 수 있는 훅. 렌더링 후에 실행된다.
  - 왜 쓰나요?
    - Side Effect를 처리하기 위해 사용한다. 외부와 상호작용하는 작업이며, 데이터 페칭, 구독, 타이머 설정, DOM 조작 등이 있다.
  - **useEffect는 정확히 언제 실행되나?**
    - 렌더링이 끝나고 커밋(화면에 그려지는 과정) 후에 실행된다.
    - **그러면 렌더링과 커밋은 정확히 어떤 과정인가?**
      - 렌더링: JSX를 통해 DOM 노드를 생성하는 과정
      - 커밋: 렌더링(DOM노드 생성) 후 DOM을 수정하는 과정. 즉, DOM 노드를 화면에 표시함.
- 클린업 함수란?
  - 리렌더링 되기 전에 실행되는 함수. 클린업 함수가 실행된 후 다시 Effect 함수가 실행된다.
  - 또, 컴포넌트가 DOM에서 제거되면 마지막으로 실행된다.
- 의존성 배열이란?
  - useEffect 콜백에서 참조된 모든 반응형 값의 목록이다.
  - 여기서 반응형 값은 props, state, 컴포넌트 본문 내부에서 직접 선언한 모든 변수와 함수를 포함한다.
  - Object.is로 이전 값과 비교한 후 변경되지 않으면 Effect를 다시 실행하지 않는다.
  - 경쟁 조건이 뭔가요?
    - 병렬 또는 동시에 실행되는 코드에서 발생하는 비정상적인 동작.
    - JavaScript 환경에서는 비동키 코드가 동시에 여러개 실행 되었을 때 가장 늦게 완료된 것이 결과 값이 되는 것을 말한다.
    - 경쟁 조건이 발생했을 때, react에서 어떻게 처리해줄 수 있나?
      - useEffect의 클린업 함수를 통해 플래그를 지정해주면 마지막에 요청한 응답만 결과 값으로 사용할 수 있다.
  - 의존성 배열이 필요없다고 react에게 증명할 때 어떤 방법이 있나?
    - 변수를 상태 값으로 만들지 않고 컴포넌트 외부로 이동 시킨다.
    - setState에 상태 값을 참조하지 않고 업데이터 함수를 넣어 호출한다.
    - 의존성 배열에 들어가는 것이 객체 혹은 배열의 식별자라면, 원시 값을 추출해 준다.
    - 객체를 useEffect 내부로 들고 들어온다.
- useEffectEvent가 뭔가요?
  - Effect 코드에서 의존성 배열에는 들어가지 않아야 하지만, 최신 반응형 값을 얻고, 싶을 때 사용한다.

---

## useLayoutEffect

- useLayoutEffect란?
  - useEffect와 비슷하다.
  - 하지만 useEffect는 브라우저의 layout, paint 이후에 비동기적으로 실행되며
  - useLayoutEffect는 브라우저의 layout이 실행되고, paint 이전에 동기적으로 실행된다.
  - 용도는?
    - DOM을 조작할 일이 있을 경우, DOM의 위치값을 계산해서 정확히 표현할 때 사용한다. useEffect를 사용하면 paint 이후에 실행되기 때문에 깜빡임이 발생하기 때문이다.

---

## useMemo

- useMemo란?
  - 리렌더링 사이의 계산 결과를 캐시할 수 있다.
  - 의존성 배열 값이 바뀌면 새로 함수를 실행해 값을 반환한다.
  - 사용 이유?
    - 계산 비용이 많은 함수의 반환 값을 메모이제이션 하여 성능을 최적화 하는데 사용한다.

---

## useCallback

- useCallback이란?
  - 리렌더링 사이의 함수 정의를 캐시할 수 있다.
- useMemo와 useCallback의 차이점은?
  - 어떤 것을 캐시할지가 다르다 memo는 함수의 반환 값이며, callback은 함수 자체이다.
