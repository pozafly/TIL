# React 렌더링 이해하기(with useState)

> [출처](https://kimyouknow.github.io/fe/React%20%EB%A0%8C%EB%8D%94%EB%A7%81%20%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0%20(with%20useState))

```jsx
import { useState } from 'react';

const add = prev => {
  console.log('add', prev);
  return prev + 1;
};

function Counter() {
  console.log('app Render');

  const [count, setCount] = useState(() => {
    console.log('init');
    return 0;
  });

  const handleClick = () => {
    console.log('click');
    setCount(add);
    setCount(add);
  };

  return <button onClick={handleClick}>count is {count}</button>;
}
```

클릭하면 2씩 올라감. 업데이터 함수를 사용했기 때문에.

**예상 동작**

```
mount: app render → init → return
click: click → add 0 → add 1 → app render → return
```

**실제 동작**

```
mount: app render → init → return
1st click: click → add 0 → app render → add 1 → return
2nd click after : click → app render → add 2 → add 3 → return
```

<img width="91" alt="스크린샷 2023-05-31 오후 5 42 14" src="https://github.com/pozafly/TIL/assets/59427983/1022857e-ecac-4458-ae0c-e49a22a561ee">

<br />

## React가 언제 또는 왜 컴포넌트를 렌더링 하는지?

React가 렌더링 하는 경우

- 컴포넌트에 예정된 상태 업데이트가 있을 경우
- 부모 컴포넌트가 렌더링 되고 해당 컴포넌트가 리렌더링에서 제외되는 기준에 충족하지 않을 경우

리렌더링 제외 조건 4가지

- 컴포넌트가 이전에 렌더링 되었어야 한다. 즉, 이미 마운트 되었어야 한다.
- 변경된 props(참조)가 없어야 한다.
- 컴포넌트에서 사용하고 있는 context 값이 변경되지 않아야 한다.
- 컴포넌트에 예정된 상태 업데이트가 없어야 한다.

**컴포넌트에 예정된 상태 업데이트가 있을 경우**. 이 경우에 오해가 있다. 상태가 변경되면 컴포넌트가 이를 감지하고, 함수형 컴포넌트가 다시 호출되는 흐름으로 이해하기 쉽다. 하지만 아님.

setState는 다르게 동작한다.

<br />

## 용어 정리

### React Element

React Element는 실제 DOM 노드가 아님. `React DOM node element` 와 `React Component element` 두 가지로 나뉜다.

- DOM element: 타입이 string일 때.
  - `<div></div>`
- Component element : 타입이 class 이거나 function일 때.
  - `<Component></Component>`

즉, type을 보고 결정한다.

### 가상돔 (Virtual DOM)

가상돔은 fiber architecture와 밀접한 관련이 있다. 가상돔이란 [가상적인 UI(an ideal, or “virtual”, representation of a UI)를 메모리에 유지하며 실제 돔과 동기화하는 프로그래밍 컨셉](https://legacy.reactjs.org/docs/faq-internals.html)이다. 그리고 이 과정을 재조정이라고 한다.

### 재조정(Reconciliation)

재조정이란, React에서 어떤 부분들이 변해야 하는지 서로 다른 두 개의 트리를 비교하는데 사용하는 알고리즘이다. 렌더링 함수를 호출할 때 가상돔을 생성하고, 이전 가상돔(snapshot)과 비교하여 변경된 부분만 실제 돔에 반영하는데, 이 때 비교하는 과정을 재조정이라고 한다.

### render phase

가상돔을 재조정하는 단계다.

### commit phase

재조정한 DOM을 실제 DOM에 적용하는 단계.

<br />

## Render와 Commit

React가 UI를 요청하고 제공하는 프로세스는 4가지 단계로 설명할 수 있다.

- **Trigger** a render
- **Rendering** the component
- **Committing** to the DOM
- Browser **paint**

### 1. Trigger a render

- createRoot으로 인한 최초 컴포넌트 렌더링
- 컴포넌트 상태를 set 함수를 활용해 업데이트

### Step 2: React renders your components

일반적으로 렌더링이라 하면, 컴포넌트 호출을 떠올린다. 하지만, Render는 가상돔 트리를 순회하며 변경된 부분을 찾고, 필요한 작업을 fiber node에 저장하는 단계.

1. React가 컴포넌트를 호출해 react element를 반환한다.
2. 가상돔 재조정 작업이 진행된다.
3. renderer가 컴포넌트 정보를 실제 DOM에 삽입한다. (mount)
4. 브라우저가 DOM을 paint한다.

**핵심은 컴포넌트 호출과 DOM 삽입은 별개다.** 그리고 DOM 삽입과 paint도 별개다.

렌더링은, 컴포넌트가 호출되어 react element를 반환하고, 가상돔에 적용하는 일련의 과정 즉, 2번까지임. 이후가 commit임.

### Step 3: React commits changes to the DOM

재조정이 끝나고, 가상돔을 실제 돔에 적용한다. 초기 렌더링의 경우, appendChild DOM api를 사용해 모든 DOM 노드를 화면에 표시한다. 리렌더링은 DOM이 최신 렌더링 출력과 일치하도록 최소한의 필수 작업을 적용한다. 차이가 있을 때만 DOM 노드를 변경한다.

commit phase에서 재조정한 가상돔을 실제 돔에 적용하고 라이프 사이클을 실행시킨다. 이때는 일관성을 위해 동기적으로 실행한다. React가 돔을 조작한 후 콜스택을 비워줘야 브라우저가 paint할 수 있기 때문.

<br />

## Snapshot

각 render의 상태는 고정되어 있다. 모든 렌더링은 고유의 prop와 state, 이벤트 핸들러를 가진다.

state는 react 내부에서 유지되고 있다. 컴포넌트를 호출할 때, 특정 렌더링에 대한 상태의 snapshot을 제공한다. 컴포넌트들은 렌더링마다 참조해야 할 상태를 가지고 있다.

<br />

## useState 동작

### Batches: [Queueing a Series of State Updates](https://react.dev/learn/queueing-a-series-of-state-updates)

react는 상태 업데이트 처리하기 전 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다. 이때, setState를 바로 실행시키지 않고 인자로 넘겨준 함수들을 큐에 넣는다. 따라서, setState를 호출하는 것은 state를 바로 업데이트 하는게 아니라 함수를 큐에 넣는 것이다.

setState가 컴포넌트 렌더링을 trigger하고 나면, render phase에서 가상돔을 재조정 한다. 이 때, 큐에 있는 update 함수들을 실행하며 새로운 상태 값을 계산한다.

따라서, 

- setState 호출 -> trigger (리렌더링 예약)
  - setState에서 업데이터 함수라면 큐에 넣음
- render phase 실행 (재조정 포함)
- 큐에 업데이터 함수가 있으면 이때 실행 -> state 계산
- 리렌더링

`setState 호출은 React에게 상태 변경을 알리는 것이지, 즉시 상태를 업데이트하지는 않는다`. React는 내부적인 변경 감지 알고리즘을 사용하여 상태 변경을 감지한다. 상태가 변경됐다면 React는 변경된 부분만 업데이트하고 필요한 경우에만 실제 돔에 반영한다. 이를 통해 React는 불필요한 렌더링 작업을 최소화하고 효율적인 UI 업데이트를 수행합니다. batching임.

아래 사진을 봐라.

<img width="108" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/aaba118d-a724-49c5-8fe0-5afdccad19af">

사실은 처음에만 Render가 한번 실행되고 나머지는 재조정 중 큐에서 실행된다.

아래 코드는 setState를 실행했을 때 실행되는 dispatchSetState함수의 일부분임. 최초 렌더링(mount) 이후 큐는 비어있기 때문에 전체 render phase 전에 다음 상태를 열심히(eagerly) 계산한다고 한다. ([소스 코드](https://github.com/facebook/react/blob/v18.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L2234-L2259))

```js
if (fiber.lanes === NoLanes && (alternate === null || alternate.lanes === NoLanes)) {  // The queue is currently empty, which means we can eagerly compute the  // next state before entering the render phase. If the new state is the  // same as the current state, we may be able to bail out entirely.  ...
```


