# useTransition

> [출처](https://1ilsang.dev/posts/js/use-transition)

useTransition은 컴포넌트 최상위 수준에서 호출되어 `startTransition` 을 통해 **우선순위가 낮은** 상태 업데이트(setState) 들을 `transition` 이라고 표시한다. 리액트는 UI 렌더링 시 우선순위에 따라 업데이트 할 수 있게 된다.

<br/>

## useTransition 이란?

`useTransition` 은 **UI를 차단하지 않고 상태를 업데이트** 할 수 있는 리액트 훅이다.

```javascript
const [isPending, startTransition] = useTransition();
```

여기서 <u>UI를 차단하지 않고</u> 라는 문구를 유의해라. useTransition을 통해 React18에 추가된 많은 기능 중 하나인 [Concurrent rendering](https://www.freecodecamp.org/korean/news/riaegteu-18yi-singineung-dongsiseong-rendeoring-concurrent-rendering-jadong-ilgwal-ceori-automatic-batching-deung/)(동시성 렌더링)을 적절하게 사용할 수 있다.

일반적으로 오래 걸리는 상태 업데이트(setState)가 존재할 경우, 업데이트가 완료된 이후에 렌더링이 일어나기 때문에 그 시간만큼 렌더 트리가 '블락(Block)' 된다. 이 때문에 유저는 아무런 동작을 할 수 없는 상태에 빠지게 되므로 UX에 좋지 않은 영향을 준다.

`startTransition` 을 통해 **우선순위가 낮은 상태 업데이트**들을 `transition` 이라고 표시해 리액트가 UI 렌더링 시 우선순위에 따라 업데이트 할 수 있도록 한다.

`transition` 으로 표시된 상태 업데이트(A라 호칭)는 다른 일반적 상태 업데이트(B)가 호출될 때 중단되고, B의 상태 업데이트가 완료된 다음 **다시** A를 렌더링 시작한다. 이를 통해 특정 컴포넌트의 렌더링이 오래 걸리더라도 다른 우선순위 높은 상태의 변경을 통해 User Interaction을 블로킹 하지 않고 자연스럽게 동작할 수 있도록 한다.

<br/>

## isPending, startTransition 이해하기

```jsx
const TabButton = ({ children, onClick }) => {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  function selectTab(nextTab) {
    startTransition(() => {
      // NOTE: async 함수는 들어오면 안된다.
      setTab(nextTab);
    });
  }
  // ...
};
```

useTransition은 두 개의 항목이 있는 배열을 반환한다.

1. `isPending` 플래그는 대기 중인 transition이 있는지 알려줌.
2. `startTransition` 함수는 상태 업데이트(setState)를 transition으로 표시해주는 함수.

**startTransition 유의 사항**

1. 동기 함수여야 한다.
2. `transition` 으로 표시된 setState는 다른 setState 업데이트 시 중단된다.
   - 다른 상태 업데이트가 있을 경우 그것을 먼저 처리한다는 뜻
3. 텍스트 입력을 제어하는데 사용할 수 있다.

<br/>

## 전체 코드 이해하기

```jsx
const App = () => {
  const [tab, setTab] = useState('about');

  return (
    <>
      {/* 탭을 클릭하면 렌더링할 탭 컴포넌트가 설정된다 */}
      <TabButton isActive={tab === 'about'} onClick={() => setTab('about')}>
        About
      </TabButton>
      <TabButton isActive={tab === 'posts'} onClick={() => setTab('posts')}>
        Posts (slow)
      </TabButton>
      <TabButton isActive={tab === 'contact'} onClick={() => setTab('contact')}>
        Contact
      </TabButton>
      <hr />
      {/* 현재 탭에 따라 탭 컴포넌트가 렌더링 된다 */}
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </>
  );
};

const TabButton = ({ children, isActive, onClick }) => {
  const [isPending, startTransition] = useTransition();

  // 현재 탭이 활성화 되면 isActive 상태가 된다.
  if (isActive) {
    return <b>{children}</b>;
  }
  // 대기 중인 transition이 있다면 isPending이 된다.
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  /**
   * props로 받은 onClick 함수를 startTransition으로 감싸주기 때문에
   * onClick 함수(setTab)은 transition으로 설정되어 렌더링시 우선순위에서 밀리게 된다.
   * 그 결과 오랜시간이 걸리는 PostsTab 컴포넌트를 렌더링 하는 도중 다른 탭을 누르게 되면
   * PostsTab 컴포넌트의 렌더링을 멈추고 다른 컴포넌트를 렌더링하게 된다.
   **/
  const handleButtonClick = () => {
    startTransition(() => {
      onClick();
    });
  };
  return <button onClick={handleButtonClick}>{children}</button>;
};

const AboutTab = () => {
  return <p>Welcome to my profile!</p>;
};
const PostsTab = () => {
  const startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // 1 ms 동안 아무것도 하지 않음으로써 매우 느린 코드를 실행한다.
  }
  return <p>PostsTab</p>;
};
const ContactTab = () => {
  return <p>ContactTab</p>;
};
const ContactTab = () => {
  return <p>ContactTab</p>;
};
```

<br/>

## Suspense와 연계하기

```jsx
const App = () => {
  return (
    <Suspense fallback={<Spinner />}>
      {/*
        위의 App 코드와 동일
       */}
    </Suspense>
  );
};
```

`startTransition` 을 `Suspense` 와 함께 사용할 경우 불필요한 로딩 인디케이터 노출을 막을 수 있다.

일반적으로 렌더링이 오래 걸리는 컴포넌트를 Suspense로 감쌀 경우 해당 컴포넌트가 렌더링 될 때마다 Suspense의 fallback 컴포넌트를 만나게 된다. 해당 서스펜스 트리 하위의 렌더링을 중단할 수 없기 때문에 오래 걸리는 렌더링을 막을 방법이 없다. 해당 렌더링이 종료될 때까지 폴백 컴포넌트를 마주해야만 한다.

이때 오래 걸리는 상태 업데이트를 startTransition으로 감싸면 `transition` 표시가 되면서 '긴급하지 않은' 상태 업데이트로 간주된다. 이로 인해 리액트는 Suspense를 통해 컨텐츠를 숨기지 않고 이전 컨텐츠를 계속 표시하게 된다.

아래와 같은 장점이 있음.

1. transition은 중단할 수 있으므로 리렌더링까지 기다릴 필요가 없다.
   - Suspense 경우 하위 컴포넌트가 모두 렌더링 될 때까지 fallback을 노출시킴
2. transition은 서스펜스 폴백을 방지(대기하지 않으므로) 하므로 갑장스러운 로딩 인디케이터 노출을 피할 수 있음.

<br/>

## 자잘 팁

### throttle, debounce 와의 차이점

이벤트 지연 및 제한은 가능하지만, UI 블로킹의 근원적 문제는 해결 불가능. 이벤트 실행 시점/횟수 줄인다 해도 한 번 실행이 되는 순간 블로킹이 되는 것은 여전하기 때문임. 근본적 원인을 해결하기 위해서는 이벤트 우선순위를 나누어 유저 인터렉션이 일어났을 때 해당 이벤트를 우선적으로 처리해 화면이 멈춘 것 처럼 보이지 않게 해야 한다.

### startTransition에 전달된 함수는 즉시 실행된다.

```js
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);

// 1, 2, 3
```

함수가 실행되는 동안 예약된 모든 상태 업데이트는 `transition` 으로 표시된다.

```js
// React 작동 방식의 간소화된 버전
let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... transition state 업데이트 예약 ...
  } else {
    // ... 긴급 state 업데이트 예약 ...
  }
}

```

transition으로 처리된 경우 transitionState로 예약(큐잉) 되고 아닌 경우 일반 state 업데이트로 예약된다.

예약된 작업들은 React18의 fiber 엔진(자체적 스케줄러를 가지고 있다)이 적절하게 스케줄링 해준다.

### useDeferredValue

useDeferredValue도 useTransition과 유사하게 낮은 우선순위를 지정하기 위한 훅이다. useTransition은 함수 실행의 우선순위를 지정하며, useDeferredValue는 값의 업데이트 우선순위를 지정한다.

<br/>

## 정리

1. useTransition은 컴포넌트 최상위 수준에서 호출되어 `startTransition`을 통해 **우선순위가 낮은** 상태 업데이트(setState)들을 `transition`이라고 표시한다. 리액트는 UI 렌더링시 우선순위에 따라 업데이트 할 수 있게 된다.
2. `startTransition` 함수는 동기 함수여야 한다.
3. `transition` 표시된 setState는 다른 setState 업데이트시 중단된다.
4. `transition` 표시된 상태 업데이트는 `Suspense`로 컨텐츠를 숨기지 않고 이전 컨텐츠를 계속 표시한다.
5. fiber 엔진을 통해 `transition`된 상태와 다른 상태의 스케줄링이 가능해졌다.


