# useMemo, useCallback

> [출처](https://medium.com/@yujso66/%EB%B2%88%EC%97%AD-usememo-%EA%B7%B8%EB%A6%AC%EA%B3%A0-usecallback-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-844620cd41a1)

리액트에서 가장 악명 높은 두 가지 훅을 이해하는 길

`useMemo`와 `useCallback

## 기본 아이디어

### useMemo

기본 아이디어는 렌더링 사이에서 계산 된 값을 '**기억**'할 수 있다는 것이다. 조금 더 풀어서 생각해야 한다. 리액트가 작동하는 방식에 대한 꽤 정교한 멘탈 모델이 필요함.

리액트가 하는 주된 일은 UI를 어플리케이션 상태와 **동기화** 하는 것이다. 이를 수행하는데 사용하는 도구를 '**리렌더링**' 이라고 한다.

각 리렌더링은 현재 어플리케이션 상태를 기반으로 하여 주어진 순간에 어플리케이션의 UI가 어떻게 보여야 하는지에 대한 스냅샷이다. 우리는 그것을 사진 더미처럼 생각할 수 있다. 각 사진은 모든 상태 변수에 특정 값이 주어졌을 때 사물이 어떻게 보이는지 포착한다.

![image](https://github.com/pozafly/TIL/assets/59427983/0a68cfe1-293a-4de9-aa6f-ffe8d1aa0d4c)

각 리렌더링은 현재 상태를 기반으로 DOM이 어떻게 생겼는지에 대한 그림을 생성한다. 위의 데모에서는 HTML로 표현되지만 실제로는 JS 객체다. 이 용어를 '가상DOM' 이라고 함.

어떤 DOM 노드를 변경해야 하는지 리액트에 직접 알려주지 않는다. 대신 리액트에 현재 상태를 기반으로 해야 하는 UI를 알려준다. 리렌더링을 통해 리액트는 새로운 스냅샷을 생성하고 '틀린 그림 찾기' 게임을 하는 것과 같이 스냅샷을 비교하여 변경해야 할 사항을 파악할 수 있다.

리액트는 기본적으로 최적화가 많이 되어 있어 일반적으로 다시 렌더링하는 것은 큰 문제가 아님. 그러나 특정 상황에서는 이런 스냅샷을 만드는데 시간이 걸린다. 이로 인해 사용자가 작업을 수행한 후 UI가 충분히 ㅃ라리 업데이트 되지 않는 것과 같은 성능 문제가 발생할 수 있다.

기본적으로 `useMemo`와 `useCallback`은 리렌더링을 최적화 하는데 도움이 되도록 구축된 도구다.

1. 주어진 렌더에서 수행해야 하는 작업의 양을 줄인다.
2. 컴포넌트가 다시 렌더링해야 하는 횟수를 줄인다.

---

<br/>

## 사용 사례1. 무거운 계산

아래 코드를 전부 읽지 말고 중요한 부분만 봐라. 아래는 소수를 찾는 컴포넌트다.

```jsx
import React from "react";

function App() {
  // 우리는 사용자가 선택한 숫자를 상태로 유지합니다.
  const [selectedNum, setSelectedNum] = React.useState(100);

  // 0과 사용자가 선택한 숫자 `selectedNum` 사이의 모든 소수를 계산합니다.
  const allPrimes = [];
  for (let counter = 2; counter < selectedNum; counter++) {
    if (isPrime(counter)) {
      allPrimes.push(counter);
    }
  }

  return (
    <>
      <form>
        <label htmlFor="num">Your number:</label>
        <input
          type="number"
          value={selectedNum}
          onChange={(event) => {
            // 컴퓨터가 과부화되는 것을 방지하기 위해, 최대 100k로 설정
            let num = Math.min(100_000, Number(event.target.value));

            setSelectedNum(num);
          }}
        />
      </form>
      <p>
        There are {allPrimes.length} prime(s) between 1 and {selectedNum}:{" "}
        <span className="prime-list">{allPrimes.join(", ")}</span>
      </p>
    </>
  );
}

// 주어진 숫자가 소수인지 여부를 계산하는 헬퍼 함수입니다.
function isPrime(n) {
  const max = Math.ceil(Math.sqrt(n));

  if (n === 2) {
    return true;
  }

  for (let counter = 2; counter <= max; counter++) {
    if (n % counter === 0) {
      return false;
    }
  }

  return true;
}

export default App;
```

- `selectedNum` 상태 값
- `for` 문을 사용해 0과 `selectedNum` 사이의 모든 소수를 수동으로 계산
- `selectedNum`을 변경할 수 있도록 숫자 input controlled(제어) component를 렌더링
- 계산한 모든 소수를 사용자에게 보여준다.

**이 코드는 상당한 양의 계산이 필요하다.** 큰 `selectedNum`을 선택하면 수만 개의 숫자를 살펴보고 소수인지 판별해야 한다.

사용자가 `selectedNum` 상태 값을 변경하면 계산을 다시 수행해야 함. 여기에 시계까지 있다고 해보자.

```jsx
// `time`은 초당 한 번 변경되는 상태 변수이므로 항상 현재 시간과 동기화됩니다.
const time = useTime();

return (
  <>
    <p className="clock">{format(time, "hh:mm:ss a")}</p>
    <form>
      // (...)
    </form>
		// (...)
  </>
);

function useTime() {
  const [time, setTime] = React.useState(new Date());

  React.useEffect(() => {
    const intervalId = window.setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => {
      window.clearInterval(intervalId);
    };
  }, []);

  return time;
}
```

이제 `selectedNum`, `time` 두 가지 상태를 갖는다. `time` 변수는 현재 시간을 반영하도록 초당 한 번 업데이트 된다.

**문제가 있다.** 상태 값 중 하나가 변경될 때마다 값 비싼 소수 계산을 모두 다시 실행한다. 그리고 `time`은 1초에 한 번 변경되기 때문에 사용자가 선택한 숫자가 변경되지 않은 경우에도 해당 소수 목록을 지속적으로 재생성한다는 의미다.

JavaScript는 싱글 쓰레드고, 매 초마다 이 코드를 계속해서 실행해 바쁘게 유지되고 있다. 하지만 이런 계산을 **'건너뛸' 수** 있다면? 이미 소수 목록이 있는 경우 매번 처음부터 계산하는 대신 그 값을 다시 사용하는 것은 어떤가?

`useMemo` 가 가능하게 해준다.

```js
const allPrimes = React.useMemo(() => {
  const result = [];
  for (let counter = 2; counter < selectedNum; counter++) {
    if (isPrime(counter)) {
      result.push(counter);
    }
  }
  return result;
}, [selectedNum]);
```

마운트 하는 동안 컴포넌트가 처음으로 렌더링 되면 리액트는 이 함수를 호출하여 모든 소수를 계산한다. 이 함수에서 반환되는 모든 것은 `allPrimes` 변수에 할당된다.

그러나 리랜더링 때 리액트는 아래와 같은 선택을 할 수 있음.

1. 함수를 다시 호출하여 값을 다시 계산
2. 이 작업을 마지막으로 수행했을 때 이미 가지고 있는 데이터를 재사용 한다.

이 질문에 답하기 위해 종속성 배열을 살펴보자. 이전 렌더링 이후 변경된 사항이 있다면 리액트는 제공된 함수를 다시 실행해 새 값을 계산한다. 그렇지 않다면 이전에 계산된 값을 재사용한다.

> ⭐️ `useMemo` 는 본질적으로 작언 캐시와 같으며 종속성은 캐시 무효화 전략이다.

이 경우 `selectedNum` 상태 값이 변경될 때만 소수 목록을 다시 계산한다. 컴포넌트가 다른 이유로 리렌더링 되는 경우(`time` 상태 값 변경) `useMemo`는 함수를 무시하고 캐시된 값을 전달한다.

이것을 메모이제이션(memoization)으로 알려져 있다. 이 훅을 `useMemo` 라고 부르는 이유다.

### 대체할 수 있는 접근 방법

정말 이게 최고 솔루션일까? 어플리케이션 구조를 재구성 해 `useMemo`의 필요성을 피할 수 있다.

```jsx
import React from "react";

import Clock from "./Clock";
import PrimeCalculator from "./PrimeCalculator";

function App() {
  return (
    <>
      <Clock />
      <PrimeCalculator />
    </>
  );
}

export default App;
```

Clock, PrimeCalculator 컴포넌트로 나누는 것임. App 에서 분기, 자체적으로 상태를 관리하는 것을 말함. 한 컴포넌트에서 리렌더링 해도 다른 컴포넌트에는 영향을 주지 않는다.

상태를 끌어 올리는 것에 대해 많이 들었지만, 때로는 상태를 아래로 push하는 것이 더 나은 방법임. 컴포넌트는 단일 책임을 가져야 하고, 완전히 관련 없는 두 가지 작업을 수행했다.

하지만, 규모가 큰 앱에서는 상태 값을 상당히 높이 올려야 하고, 아래로 밀 수 없는 상태가 있다.

### 이 상황에 다른 트릭

`PrimeCalculator` 컴포넌트 위에서 `time` 변수가 필요하다고 가정하자.

```jsx
import PrimeCalculator from "./PrimeCalculator";

// PrimeCalculator를 순수 컴포넌트로 변경
const PurePrimeCalculator = React.memo(PrimeCalculator);
```

<img width="553" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/5440da32-0235-4807-8241-1696892f4754">

`React.memo`는 컴포넌트를 감싸고 관련 없는 업데이트로부터 컴포넌트를 보호한다. `PurePrimeCalculator` 는 새로운 데이터를 수신하거나 내부 상태가 변경될 때만 다시 리렌더링 한다.

이것이 `순수 컴포넌트` 라고 함. 이 컴포넌트가 동일한 입력이 주어지면 항상 동일한 출력을 생성하고 아무 것도 변경되지 않은 경우 리렌더링을 건너 뛸 수 있다고 한다.

> 보다 전통적인 접근 방식
>
> 위의 예시는 import한 PrimeCalculator 컴포넌트에 `React.memo` 를 적용하고 있다. 이건 사실 이례적임. 실제로는 export 하는 곳에서 `React.memo`를 적용하는 경우가 많다.
>
> ```jsx
> // PrimeCalculator.js
> function PrimeCalculator() {
>   /_ Component stuff here _/;
> }
> export default React.memo(PrimeCalculator);
> ```
>
> PrimeCalculator 컴포넌트는 사용할 때 다시 고칠 필요 없이 항상 순수할 것이다.

흥미로운 **관점 전환이 있다.** 이전에는 특정 계산 결과를 메모해 소수를 계산했다. 그러나 이 경우 대신 전체 컴포넌트를 메모했다.

값비싼 계산 작업은 사용자가 새로운 `selectedNum`을 선택할 때만 다시 실행된다. 그러나 특정 느린 코드 라인보다는 상위 컴포넌트를 최적화 했다.

이제, 순수 컴포넌트를 사용하려고 시도한 적 있다면 이상한 점을 발견했을 것임. **순수 컴포넌트는 변경되지 않은 것처럼 보일 때에도 종종 꽤 자주 리렌더링 된다.**

이것은 `useMemo` 가 해결하는 두 번째 문제로 우리를 이끈다.

> 더 많은 대안
>
> Dan Abramov는 자신의 블로그 게시물 [Before you memo()](https://overreacted.io/before-you-memo/)*에서* `children`을 사용한 앱 재구성에 기반한 또 다른 접근 방식을 공유함. 메모이제이션을 수행할 필요가 없도록 함.

<br/>

## 사용 사례 2: 보존된 참조

```jsx
import Boxes from "./Boxes";

function App() {
  const [name, setName] = React.useState("");
  const [boxWidth, setBoxWidth] = React.useState(1);

  const id = React.useId();

  // 이 값 중 일부를 변경해 보세요!
  const boxes = [
    { flex: boxWidth, background: "hsl(345deg 100% 50%)" },
    { flex: 3, background: "hsl(260deg 100% 40%)" },
    { flex: 1, background: "hsl(50deg 100% 60%)" },
  ];

  return (
    <>
      <Boxes boxes={boxes} />

      <section>
        <label htmlFor={`${id}-name`}>Name:</label>
        <input
          id={`${id}-name`}
          type="text"
          value={name}
          onChange={(event) => {
            setName(event.target.value);
          }}
        />
        <label htmlFor={`${id}-box-width`}>First box width:</label>
        <input
          id={`${id}-box-width`}
          type="range"
          min={1}
          max={5}
          step={0.01}
          value={boxWidth}
          onChange={(event) => {
            setBoxWidth(Number(event.target.value));
          }}
        />
      </section>
    </>
  );
}

export default App;
```

```jsx
import React from "react";

function Boxes({ boxes }) {
  return (
    <div className="boxes-wrapper">
      {boxes.map((boxStyles, index) => (
        <div key={index} className="box" style={boxStyles} />
      ))}
    </div>
  );
}

export default React.memo(Boxes); // memo
```

Boxes는 memo로 싸여있고, 순수 컴포넌트다. 즉, props가 변경될 때마다 리렌더링 해야 함.

**그러나** 사용자가 이름을 변경할 때마다 `Boxes`도 다시 렌더링된다. 왜지?

`Boxes` 컴포넌트에는 1개의 props `boxes` 만 있으며 모든 렌더링에서 동일한 데이터를 제공하는 것처럼 보인다. `boxes` 배열에 영향을 주는 `boxWidth` 상태 변수가 있지만 변경되지 않는다.

**여기 문제가 있다.** 리렌더링 될 때마다 완전히 새로운 배열을 만들고 있다. 값만 봤을 때는 동일하지만 참조는 동일하지 않다.

`Boxes` 컴포넌트는 JavaScript 함수이기도 하다. 렌더링할 때 해당 함수를 호출한다.

```jsx
// 이 구성 요소를 렌더링할 때마다 이 함수를 호출합니다.
function App() {
  // 그리고 완전히 새로운 배열을 생성합니다.
  const boxes = [
    { flex: boxWidth, background: "hsl(345deg 100% 50%)" },
    { flex: 3, background: "hsl(260deg 100% 40%)" },
    { flex: 1, background: "hsl(50deg 100% 60%)" },
  ];

  // 이 컴포넌트에 props로 전달됩니다.
  return <Boxes boxes={boxes} />;
}
```

`name` 상태가 변경되면 `App` 컴포넌트가 다시 렌더링되어 모든 코드를 다시 실행한다. 완전히 새로운 `boxes` 배열을 구성해, `Boxes` 컴포넌트에 전달한다.

그리고 `Boxes`는 리렌더링된다. 새로운 배열을 제공했기 때문이다.

`boxes` 배열의 구조는 렌더링 간 변경되지 않았지만, 상관 없다. 리액트가 `boxes` props를 이전에 본 적이 없는 새로 생성된 배열이라 판단했다. 이 문제를 해결하기 위해 `useMemo` 훅을 사용할 수 있다.

```js
const boxes = React.useMemo(() => {
  return [
    { flex: boxWidth, background: "hsl(345deg 100% 50%)" },
    { flex: 3, background: "hsl(260deg 100% 40%)" },
    { flex: 1, background: "hsl(50deg 100% 60%)" },
  ];
}, [boxWidth]);
```

앞에서 소수를 사용하면 여기서 계산 비용이 많이 드는 계산에 대해 걱정하지 않는다. 우리의 목표는 특정 배열에 대한 **참조를 보존**하는 것이다.

사용자가 빨간색 상자의 너비를 조정할 때 `Boxes` 컴포넌트가 다시 렌더링 되기를 원하기 때문에 `boxWidth`를 종속성으로 나열한다.

![image](https://github.com/pozafly/TIL/assets/59427983/df06d900-7213-4429-91ed-ee2444197c2c)

그러나 `useMemo`를 사용하면 이전에 만든 `boxes` 배열을 다시 사용한다.

![image](https://github.com/pozafly/TIL/assets/59427983/9119c468-7360-4d0d-b86e-27dfd7c186a2)

여러 렌더링에서 동일한 참조를 유지함으로써 순수 컴포넌트가 원하는 방식으로 작동하도록 하고 UI에 영향을 주지 않는 렌더링은 무시한다.

<br/>

## useCallback

useMemo와 비교해, 배열/객체 대신 **함수**를 메모이제이션한다.

배열 및 객체와 유사하게, 함수도 값이 아닌 참조로 비교된다.

```js
const functionOne = function () {
  return 5;
};
const functionTwo = function () {
  return 5;
};

console.log(functionOne === functionTwo); // false
```

즉, 컴포넌트 내에서 함수를 정의하면 모든 단일 렌더링에서 다시 생성되어 매번 동일하지만 고유한 함수를 생성한다.

```jsx
import MegaBoost from "./MegaBoost";

function App() {
  const [count, setCount] = React.useState(0);

  function handleMegaBoost() {
    setCount((currentValue) => currentValue + 1234);
  }

  return (
    <>
      Count: {count}
      <button
        onClick={() => {
          setCount(count + 1);
        }}
      >
        Click me!
      </button>
      <MegaBoost handleClick={handleMegaBoost} />
    </>
  );
}

export default App;
```

```jsx
import React from "react";

function MegaBoost({ handleClick }) {
  console.log("Render MegaBoost");

  return (
    <button className="mega-boost-button" onClick={handleClick}>
      MEGA BOOST!
    </button>
  );
}

export default React.memo(MegaBoost);
```

`MegaBoost` 컴포넌트는 `React.memo` 덕분에 순수한 컴포넌트다. `count` 에 의존하지 않는다. 하지만, `count` 가 변경될 때마다 리렌더링 된다!

`boxes` 배열에서 봤듯, 문제는 모든 렌더에서 완전히 새로운 함수를 생성한다는 것이다. 3번 렌더링하면 `React.memo`를 뚫고 3개의 별도 `handleMegaBoost` 함수를 생성한다.

`useMemo`에 대해 배운 것을 사용해 다음과 같이 문제를 해결할 수 있다.

```js
const handleMegaBoost = React.useMemo(() => {
  return function () {
    setCount((currentValue) => currentValue + 1234);
  };
}, []);
```

배열을 반환하는 대신 함수를 반환한다. 이 함수는 `handleMegaBoost` 변수에 저장한다. 이것은 잘 작동하지만 더 나은 방법이 있다.

```js
const handleMegaBoost = React.useCallback(() => {
  setCount((currentValue) => currentValue + 1234);
}, []);
```

`useCallback`은, `useMemo` 와 같은 용도로 사용되지만, 함수를 위해 특별히 제작되었다. 함수를 직접 전달하면 해당 함수를 메모화하여 렌더링 간 스레딩 한다.

다시 말해 이 두 표현식은 같은 효과를 낸다.

```js
// 이 표현식은
React.useCallback(function helloWorld() {}, []);
// 이 표현식과 근본적으로는 같습니다.
React.useMemo(() => function helloWorld() {}, []);
```

`useCallback`은 **문법적 설탕이다.**

<br/>

## 이 훅들을 언제 사용해야 하나?

저자 개인적 생각으로는 모든 단일 객체/배열/함수에 래핑하는 것은 시간낭비다. 대부분의 경우 무시할 수 있다. 리액트는 최적화되어 있기 때문. 리렌더링은 우리가 생각하는 것만큼 느리거나 비용이 많이 들지 않음.

문제에 대한 대응에는 사용해야 함. 앱이 약간 느려지는 것을 발견하면 리액트 프로파일러를 사용해, 느린 렌더링을 추적할 수 있다. 어떤 경우, 어플을 재구성해 성능을 향상시킬 수 있다. 다른 경우는 `useMemo`, `useCallback`이 작접 속도를 높이는 데 도움이 될 수 있다.

### 일반 커스텀 훅의 내부

`useToggle`을 보자.

```jsx
function App() {
  const [isDarkMode, toggleDarkMode] = useToggle(false);

  return <button onClick={toggleDarkMode}>Toggle color theme</button>;
}
```

```js
function useToggle(initialValue) {
  const [value, setValue] = React.useState(initialValue);

  const toggle = React.useCallback(() => {
    setValue((v) => !v);
  }, []);

  return [value, toggle];
}
```

`toggle` 함수는 `useCallback` 으로 메모 되어있다.

재사용 가능한 커스텀 훅을 만들 때, **미래에 어디에 사용될지 모르기 때문**에 가능한 효율적으로 만들고 싶다. 대부분 95% 상황에서는 과할 수 있지만, 30~40번 사용하는 경우 애ㅓ플리케이션의 성능을 향상시키는 데 도움이 될 가능성이 높다.

### 내부 context provider

Context API의 provider `value`로 큰 객체를 전달하는 것이 일반적이다. 이 객체를 메모하는 것이 좋다.

```jsx
const AuthContext = React.createContext({});

function AuthProvider({ user, status, forgotPwLink, children }) {
  const memoizedValue = React.useMemo(() => {
    return {
      user,
      status,
      forgotPwLink,
    };
  }, [user, status, forgotPwLink]);

  return (
    <AuthContext.Provider value={memoizedValue}>
      {children}
    </AuthContext.Provider>
  );
}
```

이유는, Context API를 사용하는 수십 개의 순수 컴포넌트가 있을 수 있는데, `useMemo` 가 없으면 `AuthProvider` 의 부모가 다시 렌더링 되는 경우 이 모든 컴포넌트가 강제로 다시 렌더링 된다.
