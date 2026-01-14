# 동시성 렌더링(react 18)

> [출처](https://tecoble.techcourse.co.kr/post/2023-07-09-concurrent_rendering/), [출처2](https://1ilsang.dev/posts/js/use-transition)

react 18 핵심은 동시성(Concurrency)이다.

가볍게 살펴보고 `useTransition`, `useDeferredValue` 훅에 대해 알아보자.

## Concurrency?

> *Concurrency is not a feature, per se. It's a new behind-the-scenes mechanism that enables React to prepare multiple versions of your UI at the same time.*

동시성 그 자체로는 기능이 아니고, React에서 여러 버전의 UI를 동시에 준비할 수 있도록 하는 매커니즘이라는 것이다. 그렇다면 Parallelism(병렬성)과 무슨 차이가 있나?

![[assets/images/9f62c0f40b6f3374a7256a75e74cd0f0_MD5.png]]

| concurrency                 | parallelism           |
| --------------------------- | --------------------- |
| 싱글 코에서도 작동          | 멀티 코어에서만 작동  |
| 동시에 실행되는 것처럼 보임 | 실제로 동시에 실행 됨 |
| 논리적 개념                 | 물리적 개념           |

동시성은 실제로 동시에 실행되지는 않지만, CPU가 현재 실행중인 프로세스를 중단하고 다른 프로세스를 실행하는 작업인 `context switching`을 하면서 '동시에 실행되는 것처럼 보이게 하는 것'이 Concurrency다.

왜 React에서 필요하게 된걸까?

<br/>

## Blocking Rendering

react 18 이전까지 렌더링 시 발생하는 문제점이 있었다. 브라우저 메인 스레드는 싱글 스레드이기 때문에 한 번에 한 가지 작업만 할 수 있다. HTML 파싱하거나 JS 실행하거나 화면에 렌더링 하는 등, 모든 작업을 하나씩 처리해 나간다.

만약 연산이 오래 걸리는 화면을 렌더링해야 한다면 어떻게 되나?

페이지 지연이 발생, 사용자가 불편을 겪는 Blocking Rendering이 발생하게 된다.

```jsx
const App = () => {
  const [text, setText] = useState('');
  const filteredItems = filterItems(text);
  const updateFilter = e => {
    setText(e.target.value);
  };

  return (
    <div className="container">
      <input type="text" onChange={updateFilter} />
      <List items={filteredItems} />
    </div>
  );
};
```

`List`는 1부터 10000까지 숫자를 리스트로 출력하는 컴포넌트라 하자. 숫자를 입력받을 수 있고, 입력한 숫자를 포함한 것만 필터링 하여 출력한다.

따라서 입력 값이 바뀔 때마다 리스트도 업데이트 되어야 하기 때문에 새로 렌더링 해야 한다.

![[assets/images/46951a737245fe7ebed057b7d69ae2de_MD5.png]]

차이점을 확인하기 위해 크롬 개발자 도구로 CPU 성능 제한을 걸고 테스트해봤다.

![[assets/images/85a7074cecfc066b4e48891d2d86951f_MD5.gif]]

숫자를 입력하면 잇풋 창은 매우 느리게 응답한다. 이런 문제를 해결하기 위해 추가된 기능이 `useTransition` 이다.

<br/>

## useTransition, useDeferredValue

### useTransition

useTransition은, 컴포넌트 최상위 수준에서 호출되어 `startTransition`을 통해 **우선순위가 낮은** 상태 업데이트(setState)들을 `transition` 이라고 표시한다. 리액트는 UI 렌더링 시 우선순위에 따라 업데이트 할 수 있게 한다. 따라서, UI를 **차단하지 않고 상태를 업데이트** 할 수 있는 리액트 훅.

오래 걸리는 setState가 존재할 경우 업데이트가 완료된 이후에 렌더링이 일언나기 때문에 그 시간만큼 렌더 트리가 Block 된다.

```jsx
const App = () => {
  const [text, setText] = useState('');
  const [isPending, startTransition] = useTransition();
  const filteredItems = filterItems(text);

  const updateFilter = e => {
    startTransition(() => {
      setText(e.target.value);
    });
  };

  return (
    <div className="container">
      <input type="text" name="user-input" onChange={updateFilter} />
      <label htmlFor="user-input" className={isPending ? 'show' : 'hide'}>
        pending...
      </label>
      <List items={filteredItems} />
    </div>
  );
};
```

`useTransition`은 2개의 원소를 가진 배열을 반환한다. `isPending`, `startTransition` 값이다.

`startTransition`으로 `setText`를 래핑하여 상태 업데이트를 낮은 우선순위로 설정한다. 그러면 더 우선순위가 높은 이벤트(input 값 변경)가 있는 경우 `setText`는 지연 시키고 이전 값을 보여주게 된다. 또한 `isPending` 을 이용하여 형재 지연된 상태라는 것도 보여주고 있다.

![[assets/images/4980265e15d96ee433549b8bfb62513e_MD5.gif]]

`useTransition`을 적용하니 리스트 렌더링과는 별개로, 인풋 창 반응이 빨라졌다.

> 📌 startTransition 유의 사항
>
> 1. 동기 함수여야 한다.
> 2. `transition` 으로 표시된 setState는 다른 setState 업데이트 시 중단된다.
>    - 다른 상태 업데이트가 있을 경우 그것을 먼저 처리한다는 뜻
> 3. 텍스트 입력을 제어하는데 사용할 수 없다.

### useDeferredValue

이를 통해 조금 더 나은 사용자 경험을 제공할 수 있다. 근데 비슷한 훅이 하나 더 있음. `useDeferredValue`. `useTransition` 훅과 비슷하게 우선순위를 낮춰 Concurrent features를 사용할 수 있다.

차이점으로는 '상태를 업데이트 하는 코드'를 래핑하여 우선순위를 낮추는 `useTransition` 와 달리 `useDeferredValue` 는 '값'의 업데이트 우선순위를 낮춘다.

또한 상태를 props로 받는 등 제어할 수 없을 겨우 사용하는 것을 권장하고 있다.

```jsx
const List = ({ items }) => {
  return (
    <ul>
      {items.map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
};
```

```jsx
const List = ({ items }) => {
  const deferredItems = useDeferredValue(items);
  return (
    <ul>
      {deferredItems.map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
};
```

`List` 컴포넌트에 적용할 수 있다. 기존 props로 받던 `items`를 `useDeferredValue`로 래핑했다. 이후 `App` 에서 `useTransition`을 사용하지 않고 실행해도, 결과적으로 `startTransition`을 사용했을 때와 유사한 결과를 얻을 수 있다.

<br/>

## Debounce, throttle

debounce, throttle과 비슷하다. 다만 두 방법 모두 Blocking Rendering을 해결하기는 아쉬운 점이 있다. 여러 번 발생하는 이벤트에서 가장 마지막 또는 제일 처음 이벤트만을 실행하도록 하는 debounce와, 여러 번 발생하는 이벤트를 일정 시간 동안 한 번만 실행되도록 만드는 throttle 모두 개발자가 timeout을 지정해주어야 한다.

즉, 만약 사용자 기기 성능은 뛰어난데 timeout이 길게 설정되어 오래 기다리게 되거나, 반대로 오래된 기기를 사용하는데 너무 자주 이벤트가 실행되어 화면이 버벅대면, 사용자는 불쾌한 경험을 하게 된다는 것이다.

그러므로 concurrent features를 잘 활용하면, debounce, throttle을 대체하거나, 함께 사용해 좋은 사용자 경험을 제공할 수 있다.
