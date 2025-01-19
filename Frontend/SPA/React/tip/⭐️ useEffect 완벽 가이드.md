# useEffect 완벽 가이드

> 출처: https://rinae.dev/posts/a-complete-guide-to-useeffect-ko?fbclid=IwAR1zptywA5R9L9bOXvChPBL8uRgOqaCbkeaBaQ2-HN7KgLHBmrUzyAgyr3Y

아래와 같은 질문을 할 수 있다.

- `useEffect` 로 `componentDidMount` 동작을 흉내내려면 어떻게 하나?
- `useEffect` 안에서 데이터 data fetching은 어떻게 해야함? 두번째 인자로 오는 배열(`[]`)은 뭘까?
- 이펙트를 일으키는 의존성 배열에 함수를 명시해도 되나?
- 왜 가끔씩 data fetching이 무한루프에 빠지는 걸까?
- 왜 가끔씩 이펙트 안에서 이전 state나 prop 값을 참조할까?

<br/>

## TLDR (Too Long; Didn't Read - 요약)

🤔 **질문: `useEffect` 로 `componentDidMount` 동작을 흉내내려면 어떻게 하지?**

완전히 같진 않지만 `useEffect(fn, [])` 으로 가능하다. `componentDidMount` 와 달리 prop과 state를 잡아둘 것이다. 그래서 콜백 안에서도 초기 prop과 state를 확인할 수 있다. 만약 **최신의** 상태 등을 원한다면, ref에다 담아둘 수는 있다. 하지만 보통 더 간단하게 코드를 구조화하는 방법이 있기 때문에 굳이 이 방법을 쓸 필요도 없음. 이펙트를 다루는 멘탈 모델은 `componentDidMount` 나 다른 라이프사이클 메서드와 다르다는 점을 인지해야 함. 그리고 어떤 라이프사이클 메서드와 비슷한 동작을 하도록 만드는 방법을 찾으려고 하면 오히려 혼란을 더 키울 뿐임. 더 생산적으로 접근하기 위해 '이펙트 기준으로 생각해야(thinking in effects)' 하며 이 멘탈 모델은 동기화를 구현하는 것에 가깝지 라이프사이클 이벤트에 응답하는 것과는 다르다.

🤔 **질문: `useEffect` 안에서 데이터 페칭은 어떻게 해야할까? 두번째 인자로 오는 배열(`[]`) 은 뭐지?**

[이 링크의 글이](https://www.robinwieruch.de/react-hooks-fetch-data/) `useEffect` 를 사용하여 데이터를 불러오는 방법을 파악하는데 좋은 기본서가 된다. 글을 꼭 끝까지 읽어보자! 지금 읽고 계시는 글처럼 길지 않다. `[]` 는 이펙트에 리액트 데이터 흐름에 관여하는 어떠한 값도 사용하지 않겠다는 뜻. 그래서 한 번 적용되어도 안전하다는 뜻이기도 함. 이 빈 배열은 실제로 값이 사용되어야 할 때 버그를 일으키는 주된 원인 중 하나다. 잘못된 방식으로 의존성 체크를 생략하는 것 보다 의존성을 필요로 하는 상황을 제거하는 몇 가지 전략을(주로 `useReducer`, `useCallback`) 익혀야 할 필요가 있다.

🤔 **질문: 이펙트를 일으키는 의존성 배열에 함수를 명시해도 되는걸까?**

추천하는 방법은 prop이나 state를 반드시 요구하지 않는 함수는 컴포넌트 바깥에 선언해서 호이스팅하고, 이펙트 안에서만 사용되는 함수는 이펙트 함수 내부에 선언하는 것임. 그러고 나서 만약에 랜더 범위 안에 있는 함수를 이펙트가 사용하고 있다면 (prop으로 내려오는 함수 포함해서), 구현부를 `useCallback` 으로 감싸자. 왜 이런걸 신경써야 함? 함수는 prop과 state로부터 값을 '볼 수' 있다. 그러므로 리액트의 데이터 플로우와 연관이 있지요. [자세한 답변](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)은 훅 FAQ 부분에 있다.

🤔 **질문: 왜 가끔씩 데이터 페칭이 무한루프에 빠지는걸까?**

이펙트 안에서 데이터 페칭을 할 때 두 번째 인자로 의존성 배열을 전달하지 않았을 때 생길 수 있는 문제. 이게 없으면 이펙트는 매 랜더마다 실행된다. 그리고 state를 설정하는 일은 또 다시 이펙트를 실행한다. 의존성 배열에 항상 바뀌는 값을 지정해 두는 경우에도 무한 루프가 생길 수 있다. 하나씩 지워보면서 어느 값이 문제인지 확인할 수도 있지만, 사용하고 있는 의존 값을 지우는 일은(아니면 맹목적으로 `[]` 을 지정하는 것은) 보통 잘못된 해결법. 그 대신 문제의 근원을 파악하여 해결해야 함. 예를 들어 함수가 문제를 일으킬 수 있다. 그렇다면 이펙트 함수 안에 집어넣거나, 함수를 꺼내서 호이스팅 하거나, `useCallback` 으로 감싸서 해결할 수 있다. 객체가 재생성되는 것을 막으려면 `useMemo` 를 비슷한 용도로 사용할 수 있다.

🤔 **질문: 왜 가끔씩 이펙트 안에서 이전 state나 prop 값을 참조할까?**

이펙트는 언제나 자신이 정의된 블록 안에서 랜더링이 일어날 때마다 prop과 state를 "지켜본다". 이렇게 하면 [버그를 방지할 수 있지만](https://overreacted.io/ko/how-are-function-components-different-from-classes/) 어떤 경우에는 짜증날 수 있다. 그럴 때는 명시적으로 어떤 값을 가변성 ref에 넣어서 관리할 수 있다(링크에 있는 글 말미에 설명되어 있다). 혹시 기대한 것과 달리 이전에 랜더링될 때의 prop이나 state가 보인다면, 아마도 의존성 배열에 값을 지정하는 것을 깜빡했을 거임. 이 [린트 규칙](https://github.com/facebook/react/issues/14920)을 사용하여 그 값을 파악할 수 있도록 연습해 보자. 며칠 안으로 자연스레 몸에 밸 것. 또한 FAQ 문서에서 [이 답변 부분을](https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function) 읽어보자.

------

useEffect를 **완전히** 이해해보자.

<br/>

## 모든 랜더링은 고유의 Prop와 State가 있다

이펙트를 이해하기 전, 렌더링에 대해 이야기 해야 함.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>You Clicked {count} times</p>   // 📌
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

`p 태그` 📌를 자세히 보자. count가 state의 변화를 관찰해 자동으로 업데이트 한다는걸까? 정확한 [정확한 멘탈 모델](https://overreacted.io/ko/react-as-a-ui-runtime/)은 아니다. `count`는 단지 숫자다. 마법의 '데이터 바인딩' 이나 '워쳐' 나 '프록시', 그 어떤 것도 아니다. 아래와 같이 단순한 숫자.

```jsx
const count = 42;
// ...
<p>You Clicked {count} times</p>
// ...
```

처음으로 컴포넌트가 렌더링 될 때, `useState`로 부터 가져온 `count`변수는 0이다. setCount(1) 을 호출하면, 다시 컴포넌트를 호출하고. 이때 `count`는 `1` 이 되는 식.

```jsx
// 처음 랜더링 시
function Counter() {
  const count = 0; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  const count = 1; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  const count = 2; // useState() 로부터 리턴
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```

state를 업데이트 할 때마다, 리액트는 컴포넌트를 호출한다. 매 랜더 결과물은 고유의 `count` 상태 값을 **살펴본다**. 그리고 이 값은 함수 안에 *상수*로 존재하는 값이다.

**따라서 위에 표시한 줄은 어떠한 데이터 바인딩도 수행하지 않는다.**

```jsx
<p>You Clicked {count} times</p>
```

이것은 렌더링 결과물에 숫자 값을 내장하는 것에 불과하다. 이 숫자 값은 리액트를 통해 제공된다. `setCount`를 호출할 때, 리액트는 다른 `count` 값과 함께 컴포넌트를 다시 호출한다. 그러면 리액트는 가장 최신의 렌더링 결과물과 일치하도록 DOM 을 업데이트 한다.

알아야 할 점은 특정 랜더링 시 그 안에 있는 `count` 상수는 시간이 지난다고 바뀌는 것이 아니라는 점. 컴포넌트가 다시 호출되고, 각각의 렌더링마다 격리된 고유의 `count` 값을 **보는** 것이다.

(이 과정을 자세히 알고 싶다면, 저자가 쓴 글 [UI 런타임으로서의 React](https://overreacted.io/ko/react-as-a-ui-runtime/)를 읽어보자.)

<br/>

## 모든 렌더링은 고유의 이벤트 핸들러를 가진다

이벤트 핸들러의 경우는 어떨까? 예제를 보자. 이 컴포넌트는 3초 뒤에 `count` 값과 함께 alert를 띄워준다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

단계별로 실행한다고 해보자.

- 카운터를 3번 눌러 3으로 증가
- Show alert을 누른다
- 타암아웃이 실행되기 전, 카운터를 5로 증가

직접 해보자. [저자가 만든 코드](https://codesandbox.io/s/w2wxl3yo0l). 실직적인 예시를 들자면 채팅 앱에서 state에 현재 수신자 ID를 가지고 있는 상태에서 전송 버튼을 누른 것이나 마찬가지. 왜 결과 값으로 3이 나왔는지는 [이 글이](https://overreacted.io/ko/how-are-function-components-different-from-classes/) 자세히 설명해 준다.

alert은 버튼을 클릭할 때의 state를 잡아둘 것이다. 왜? 위에서 `count` 값이 매번 별개의 함수 호출마다 존재하는 상수 값이라는 이야기를 했다. **함수는 여러번 호출되지만(렌더링마다 함번씩), 각각의 렌더링에서 함수 안의 count 값은 상수이자 독립적인 값(특정 렌더링 시의 상태)으로 존재한다.**

리액트만 특별히 이렇게 동작하는게 아니다. 보통의 함수도 같은 방식으로 동작한다.

```js
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert('Hello, ' + name);
  }, 3000);
}

let someone = { name: 'Dan' };
sayHi(someone);

someone = { name: 'Yuzhi' };
sayHi(someone);

someone = { name: 'Dominic' };
sayHi(someone);
```

여기서 외부의 someone 변수는 여러번 재할당 된다. (리액트 어딘가에서 *현재의* 컴포넌트 상태가 바뀔 수 있는 것처럼.) 하지만 sayHi 내부에서, 특정 호출마다 person과 엮여있는 name이라는 지역 상수가 존재한다. 이 상수는 지역 상수이기 때문에, 각각의 함수 호출로부터 분리되어 있다! 결과적으로 타임아웃 이벤트가 실행될 때마다 각자의 얼럿을 고유의 `name`을 기억하게 되는 것이다!

> 여기서 알 수 있는 것은 지역상수가 각자 다른 객체라는 것. 따라서 리액트는 불변성을 유지하는구나.

이 설명을 통해 클릭 시 우리의 이벤트 핸들러가 어떻게 `count` 값을 잡아두었는지 알게 됨. 아래와 같이 치환해보면, 매 랜더링 마다 각각의 `count` 값을 '보는' 것이다.

```jsx
// 처음 랜더링 시
function Counter() {
  const count = 0; // useState() 로부터 리턴
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  const count = 1; // useState() 로부터 리턴
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  const count = 2; // useState() 로부터 리턴
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}
```

따라서 효과적으로, 각각의 렌더링은 고유한 **버전**의 `handleAlertClick`을 리턴한다. 각각의 버전은 고유의 count를 `기억한다`.

```js
// 처음 랜더링 시
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 0);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // 0이 안에 들어있음
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 1);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // 1이 안에 들어있음
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 2);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // 2가 안에 들어있음
  // ...
}
```

따라서 이벤트 핸들러가 특정 렌더링에 '속해있으며', 얼럿 버튼을 클릭할 때 *그 랜더링 시점*의 counter state를 유지한 채로 사용하는 것이다.

**특정 렌더링 시 그 내부에서 props와 state는 영원히 같은 상태로 유지**된다. props와 state가 렌더링으로부터 분리되어있다면, 이를 사용하는 어떠한 값(이벤트 핸들러를 포함하여)도 분리되어 있는 것이다. 이들도 마찬가지로 특정 렌더링에 *속해있다*. 따라서 이벤트 핸들러 내부의 비동기 함수라 할지라도 같은 count 값을 '보게'될 것이다.

*참고 노트: 위의 예제에서 handleAlertClick 함수 안에 직접 구체적인 count 값을 넣어 표시 했다. count는 각각의 렌더링 내부에서 변할 수 없는 값, 즉, const로 선정되어있는 숫자 값이기 때문. 객체를 생각해보면 오로지 불변 값을 사용한다는 전제가 있어야 함. setSomething(newObj)를 호출할 때 객체를 변형하지 않고 새 객체를 전달하는 것처럼. 이전 렌더링에 속해 있는 state는 손상되지 않기 때문.*

<br/>

## 모든 랜더링은 고유의 이펙트를 가진다

이펙트라고 크게 다르지 않다. [이 문서](https://reactjs.org/docs/hooks-effect.html)에 있는 예제를 보자.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

누를 때마다 title이 변화함. 어떻게 이펙트가 최신의 count 상태를 읽어들일까?

아마도 뭔가 '데이터 바인딩' 이라던가 '옵저버' 라던가 하는게 이펙트 함수 안에서 `count`의 업데이트를 반영할까? 아니면 count는 가변 값이라 리액트가 컴포넌트 안에 값을 지정해주고 이펙트가 항상 최신 값을 바라볼 수 있게 해 주는 걸까?

아니다.

이미 count는 특정 컴포넌트 렌더링에 포함되는 `상수`라고 배웠다. 이벤트 핸들러는 그 랜더링에 '속한' count 상태를 `본다`. count는 특정 범위 안에 속하는 변수이기 때문. 이펙트도 똑같다.

**'변화하지 않는' 이펙트 안에서 count 변수가 임의로 바뀐다는게 아니다. 이펙트 함수 자체가 매 렌더링마다 별도로 존재한다.** 각각의 이펙트 버전은 매번 랜더링에 '속한' count 값을 '본다'.

```js
// 최초 랜더링 시
function Counter() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${0} times`;
    }
  );
  // ...
}
// 클릭하면 함수가 다시 호출된다
function Counter() {
  // ...
  useEffect(
    // 두 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${1} times`;
    }
  );
  // ...
}
// 또 한번 클릭하면, 다시 함수가 호출된다
function Counter() {
  // ...
  useEffect(
    // 세 번째 랜더링의 이펙트 함수
    () => {
      document.title = `You clicked ${2} times`;
    }
  );
  // ..
}
```

리액트는 이펙트 함수를 기억해놨다가 DOM의 변화를 처리하고 브라우저가 스크린에 그리고 난 뒤 실행한다.

따라서 비록 우리는 하나의 개념으로 이펙트를 이야기 하고 있지만(도큐먼트의 타이틀을 업뎃하는 동작), 사실 매 랜더링마다 `다른 함수`라는 뜻. 그리고 각각의 이펙트 함수는 그 랜더링에 '속한' props, state를 '본다'.

**개념적으로, 이펙트는 랜더링 결과의 일부라 생각할 수 있다.**

하지만, 엄격하게는 아니다. 우리가 생각하는 멘탈 모델 속에서 이펙트 함수는 이벤트 핸들러처럼 특정 렌더링에 속하는 함수라고 생각하면 된다.

---

더 자세하게 이해할 수 있도록, 첫 번째 랜더링을 되짚어 보자.

- 리액트: state가 `0`일 때의 UI를 보여줘
- 컴포넌트
  - 여기 랜더링 결과물로 `<p>You clicked 0 times</p>` 가 있어.
  - 그리고 모든 처리가 끝나고 이 이펙트를 실행하는 것을 잊지마: `() => { document.title = 'You clicked 0 times' }`
- 리액트: 좋아. UI를 업뎃하겠어. 브라우저, 나 DOM에 뭘 좀 추가하려 해.
- 브라우저: 좋아, 화면에 그려줄게.
- 리액트: 좋아 이제 컴포넌트 니가 준 이펙트를 실행할거야.
  - `() => { document.title = 'You clicked 0 times' }` 를 실행하는 중.

---

그럼 버튼을 클릭하면 어떤 일이 벌어지는지 복습해보자.

- 컴포넌트: 이봐 리액트, 내 상태를 `1` 로 변경해줘.
- 리액트: 상태가 `1`일 때의 UI를 줘.
- 컴포넌트
  - 여기 렌더링 걸과물로 `<p>You clicked 1 times</p>` 가 있어.
  - 그리고 모든 처리가 끝나고 이 이펙트를 실행하는 것을 잊지마: `() => { document.title = 'You clicked 1 times' }`
- 리액트: 좋아. UI를 업뎃하겠어. 브라우저, 나 DOM에 뭘 좀 추가하려 해.
- 브라우저: 좋아, 화면에 그려줄게.
- 리액트: 좋아 이제 컴포넌트 니가 준 이펙트를 실행할거야.
  - `() => { document.title = 'You clicked 1 times' }` 를 실행하는 중.

<br/>

## 모든 랜더링은 고유의 … 모든 것을 가지고 있다

이제 이펙트는 매 랜더링 후 실행되며, 그리고 개념적으로 컴포넌트 결과물의 일부로서 특정 랜더링 시점의 props와 state를 '본다'는 것을 알았다. 이게 맞는지 실험해보자.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

약간 뜸을 들이며 버튼을 여러번 누르면 어떤 일이 일어날까?

순서대로 찍히지 않고 무시된 채로 찍힐 것이라 예상하지만 아니다. 로그는 순서대로 잘 출력된다. 각각의 타이머는 특정 랜더링에 속하기 때문에 그 시점의 `count` 값을 가진다. [여기 코드가 있음.](https://codesandbox.io/s/lyx20m1ol)

하지만, [클래스 컴포넌트로 만들면](https://codesandbox.io/s/kkymzwjqz3) 이렇게 동작하지 않는다. 이 링크의 클래스 컴포넌트도 함수 컴포넌트와 같은 동작을 하리라고 착각하기 쉽다.

```js
componentDidUpdate() {
  setTimeout(() => {
    console.log(`You clicked ${this.state.count} times`);
  }, 3000);
}
```

하지만 `this.state.count` 값은 특정 랜더링 시점의 값이 아니라 언제나 최신의 값을 가리킨다. 버튼을 5번 눌렀다면, 매번 5가 찍혀있는 로그를 보게 된다.

> 내가 알고 있는 바로는 class 형 컴포넌트에서는 render() 함수 이하에 있는 것들이 랜더링 시 새로 만들어진다고 알고 있다. 따라서
>
> - class 형 컴포넌트의 `state`는 render() 함수 위에 선언되어져있고,
> - functional 형 컴포넌트의 state는 함수형 컴포넌트 자체에 속해있어 매번 랜더링 시 새로 만들어지는 개념이므로?

저자는 훅이 자바스크립트 클로저에 아주 많이 의존하고 있다는게 아이러니 하다 생각함.(저자가 왜 아이러니 하다고 했는지 모르겠다. 원본도 찾아봤는데 정말 아이러니하다고 표현했음. 클로저에 의존하면 좋은거 아냐?) 이에 반해 클래스 컴포넌트는 전통적으로 타임아웃에서 잘못된 값을 가져오는 문제에 시달리고 있는데 이 문제는 주로 클로저와 연관이 있다. 왜냐면 이 예시에서 발생한 진짜 문제의 근원은 클로저 자체가 아니라 가변값 변경(mutation)이기 때문임. (리액트는 `this.state`가 최신 상태를 가리키도록 변경한다.)

**클로저는 접근하려는 값이 절대 바뀌지 않을 때 유용하다. 반드시 상수를 참조하고 있기 때문에 생각을 하기 쉽도록 만들어준다.** 그리고 위에서 말한대로 props와 state는 특정 랜더링 안에서 절대 바뀌지 않는다. 클로저를 사용해 위의 클래스 컴포넌트 예제를 고칠 수 있다.

```js
componentDidMount() {
  const count = this.state.count;   // 프라이빗 변수 추가
  setTimeout(() => {
    console.log(`You clicked ${count} times`);
  }, 3000);
}
```

[클로저를 이용한 class 형 컴포넌트 예제](https://codesandbox.io/s/w7vjo07055)

<br/>

## 흐름을 거슬러 올라가기

이 시점에서 다시 한번 생각해보자.

> "컴포넌트의 랜더링 안에 있는 **모든**함수는 (이벤트 핸들러, 이펙트, 타임아웃이나 그 안에서 **호출**되는 API 등) 랜더(render)가 호출될 때 정의된 props와 state 값을 잡아둔다."

따라서 밑의 두 예제는 같은 것임.

```js
function Example(props) {
  useEffect(() => {
    setTimeout(() => {
      console.log(props.counter);
    }, 1000);
  });
  // ...
}
```

```js
function Example(props) {
  const counter = props.counter;
  useEffect(() => {
    setTimeout(() => {
      console.log(counter);
    }, 1000);
  });
  // ...
}
```

**props나 state를 컴포넌트 안에서 일찍 읽어 들였는지 아닌지는 상관 없음.** 그 값들은 바뀌지 않을 것이니까! 하나의 랜더링 스코프 안에서 props와 state는 변하지 않은 값으로 남아있다. (값들을 분해 할당하면 더 확실해짐.)

때때로 이펙트 안에 정의해둔 콜백에서 사전에 잡아둔 값을 쓰는게 아니라 최신의 값을 이용하고 *싶을 때*가 있다. 제일 쉬운 방법은 ref를 이용하는 것임.

과거의 랜더링 시점에서 *미래의* props나 state를 조회할 필요가 있을 때 주의해야할 점은, 이런 방식은 흐름을 거슬러 올라가는 일이라는 것이다. 잘못되진 않았지만(어떤 경우에는 반드시 필요함) 패러다임에서 벗어나는게 덜 '깨끗해' 보일 수 있다. 아래의 코드는 의도적인 결과인데 주석으로 표시 해둔(📌) 코드는 타이밍에 민감하고 다루기 어렵기 때문이다. 클래스 컴포넌트라면 언제 값을 조회하는지 덜 명확하다.

클래스 컴포넌트로 일으켰던 동작을 따라해본 버전의 [카운터 예제](https://codesandbox.io/s/rm7z22qnlp)가 있다.

```js
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);   // 📌
  useEffect(() => {
    // 변경 가능한 값을 최신으로 설정한다
    latestCount.current = count;   // 📌
    setTimeout(() => {    // 📌
      // 변경 가능한 최신의 값을 읽어 들인다
      console.log(`You clicked ${latestCount.current} times`);   // 📌
    }, 3000);   // 📌
  });
  // ...
}
```

이렇게 해주면, ref를 사용하면, 3초 안에 5번을 클릭했을 때, 마지막 값인 5가 log에 5번 찍히는 것을 볼 수 있다. 즉, 최신 state 값을 가져온 것임.

리액트로 어떠한 값을 직접 변경하는게 꺼림칙하지만, 리액트의 클래스 컴포넌트는 정확하게 이런 식으로 `this.state`를 재할당하고 있다. 미리 잡아둔 props 및 state와는 달리 특정 콜백에서 `latestCount.current`의 값을 읽어 들일 때 언제나 같은 값을 보장하지 않는다. 정의된 바에 따라 이 값은 언제나 변경할 수 있다. 그렇기 때문에 이런 사용 방법은 기본 동작이 아니며, 우리가 직접 가져다 써야 함.

<br/>

## 그러면 클린업(cleanup)은 뭐지?

[문서에서 설명한 대로](https://reactjs.org/docs/hooks-effect.html#effects-with-cleanup), 어떤 이펙트는 클린업 단계를 가질 수도 있음. 본질적으로 클린업의 목적은 구독과 같은 이펙트를 "되돌리는" 것이다.

```js
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
  };
});
```

첫번째 랜더링에서 `props`이 `{ id: 10 }` 이고, 두번째 랜더링에서 `{ id: 20 }` 이라고 해보자. *아마도* 아래와 같은 흐름으로 흘러갈 것이라 생각할 거임.

- 리액트가 `{ id: 10 }` 을 다루는 이펙트를 클린업 한다.
- 리액트가 `{ id: 20 }` 으로 UI를 렌더링 한다.
- 리액트가 `{ id: 20 }` 으로 이펙트를 실행한다.

(실제로는 조금 다르다)

위의 멘탈 모델대로라면, 클린업이 리랜더링 되기 전에 실행되고 이전의 prop을 '보고', 그 다음 새 이팩트가 리랜더링 이후 실행되기 때문에 새 prop을 '본다고' 생각할 수 있음. 이 멘탈 모델은 클래스의 라이프사이클(`componentWillUnmount`)을 그대로 옮겨놓은 것과 같고, 실제로 여기서는 잘못된 내용이다. 왜?

리액트는 브라우저가 페인트 하고 난 뒤에야 이펙트를 실행한다. 이렇게 해서 대부분의 이펙트가 스크린 업데이트를 가로막지 않기 때문에 앱을 빠르게 만들어준다. 마찬가지로 이펙트의 클린업도 미뤄진다. **이전 이펙트는 새 prop과 함께 리랜더링 되고 난 '뒤에' 클린업 된다.**

- 리액트가 `{ id: 20 }` 을 가지고 UI를 랜더링한다.
- 브라우저가 실제 그리기를 한다. 화면 상에서 `{ id: 20 }` 이 반영된 UI를 볼 수 있다.
- 리액트는 `{ id: 10 }` 에 대한 이펙트를 클린업한다.
- 리액트가 `{ id: 20 }` 에 대한 이펙트를 실행한다.

이상하게 느낄 수 있다. 그런데 어떻게 prop이 `{ id: 20 }` 으로 바뀌고 나서도 이전 이펙트의 클린업이 여전히 예전 값인 `{ id: 10 }` 을 '보는' 걸까?

이전 단락을 인용해보면,

> 컴포넌트가 랜더링 안에 있는 `모든` 함수는 (이벤트 핸들러, 이펙트, 타임아웃이나 그 안에 호출되는 API등) 랜더가 호출될 때 정의된 props와 state 값을 잡아둔다.

**이제 답이 명확해졌다. 어찌되었건 이펙트의 클린업은 '최신' prop을 읽지 않는다. 클린업이 정의된 시점의 랜더링에 있던 값을 읽는 것이다.**

```js
// 첫 번째 랜더링, props는 { id: 10 }
function Example() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트
    () => {
      ChatAPI.subscribeToFriendStatus(10, handleStatusChange);
      // 첫 번째 랜더링의 클린업
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(10, handleStatusChange);
      };
    }
  );
  // ...
}
// 다음 랜더링, props는 { id: 20 }
function Example() {
  // ...
  useEffect(
    // 두 번째 랜더링의 이펙트
    () => {
      ChatAPI.subscribeToFriendStatus(20, handleStatusChange);
      // 두 번째 랜더링의 클린업
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(20, handleStatusChange);
      };
    }
  );
  // ...
}
```

이해 안가는데.. 몬말인지.. ㅜㅜ

그 어떠한 것도 첫 번째 랜더링의 클린업이 바라보면 값을 `{ id: 10 }` 이외의 것으로 만들 수 없다. 이렇게 리액트는 페인팅 이후 이펙트를 다루는게 기본이고, 그 결과 앱을 빠르게 만들어준다. 이전 props는 우리의 코드가 원한다면 남아있다.

<br/>

## 라이프사이클이 아니라 동기화

저자가 리액트를 좋아하는 것 중 하나는 처음 랜더링 결과물과 그 업데이트를 통합하여 표현하고 있다는 점. 이로 인해 우리의 프로그램의 [엔트로피를 줄일 수 있다.](https://overreacted.io/the-bug-o-notation/)

컴포넌트가 이렇게 생겼다고 해보자.

```jsx
function Greeting({ name }) {
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

맨처음 `<Greeting name="Dan" />` 을 랜더링 한 다음에 바로 이어서 `<Greeting name="Yuzhi" />` 를 랜더링하던지, 아예 `<Greeting name="Yuzhi" />` 만 랜더링하던지 모든 경우의 결과는 `Hello, Yuzhi` 로 같다.

"모든 것은 여정에 달렸지 목적지에 달린것이 아니다" 라는 말은 리액트에서는 정 반대로 적용된다. 즉, **"모든 것은 목적지에 달렸지 여정에 달린게 아니다."** 리액트는 우리가 지정한 props와 state에 따라 DOM과 `동기화` 한다. 랜더링 시 '마운트'와 '업데이트'의 구분이 없다.

이펙트도 같은 방식이다. `useEffect`는 리액트 트리 바깥에 있는 것들을 props와 state에 따라 *동기화*할 수 있게 한다.

```js
function Greeting({ name }) {
  useEffect(() => {
    document.title = 'Hello, ' + name;
  });
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

위 코드는 친숙한 *마운트 / 업데이트 / 언마운트* 멘탈 모델과는 묘하게 다르다. 다르다는 것을 마음에 담아두는게 중요하다. **만약 컴포넌트가 첫 번째로 랜더링 할 때와 그 후에 다르게 동작하는 이펙트를 작성하려고 한다면, 흐름을 거스르는 것!** 랜더링 결과물이 '목적지'에 따라가는 것이 아니라, '여정'에 따른다면 동기화에 실패한다.

컴포넌트를 prop A, B, C 순서대로 랜더링 하던지, 바로 C로 랜더링 하던지 별로 신경쓰이지 않아야 한다. 잠깐 차이가 있을 수 있지만(예: 데이터를 불러올 때), 결국 마지막 결과물은 같아야 함.

당연하지만 여전히 모든 이펙트를 *매번* 랜더링마다 실행하는 것은 효율이 떨어질 수 있다. (그리고 어떤 경우는 무한 루프를 일으킬 수 있다.) 그러면 어떻게 해결하나?

<br>

## 리액트에게 이펙트를 비교하는 법 가르치기

우리는 이미 DOM 자체를 알고 있다. 매번의 리랜더링마다 DOM 전체를 새로 그리는게 아니라 리액트가 실제로 바뀐 부분만 DOM을 업뎃 함.

```jsx
<h1 className="Greeting">
  Hello, Dan
</h1>

// 위에 것을 아래 것으로 이렇게 바꾼다면

<h1 className="Greeting">
  Hello, Yuzhi
</h1>
```

리액트는 두 객체를 비교한다.

```js
const oldProps = { className: 'Greeting', children: 'Hello, Dan' };
const newProps = { className: 'Greeting', children: 'Hello, Yuzhi' };
```

각각의 prop을 짚어보고 `children`이 바뀌어 DOM 업데이트가 필요하다고 파악했지만, `className`은 그렇지 않다. 그래서 그저 아래의 코드만 호출된다.

```js
domNode.innerText = 'Hello, Yuzhi';
// domNode.className은 건드릴 필요 없음
```

이펙트에도 이런 방법을 적용할 수 있을까? 이펙트를 적용할 필요가 없다면 다시 실행하지 않는 것이 좋을텐데, 예를 들어 아래의 컴포넌트는 상태 변화 때문에 다시 랜더링 될 것이다.

```jsx
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);
  
  useEffect(() => {
    document.title = 'Hello, ' + name;
  });
  
  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>
        Increment
      </button>
    </h1>
  );
}
```

하지만 이펙트는 `counter` 상태 값을 사용하지 않는다. 이펙트는 `document.title`과 `name` prop을 동기화 하지만, `name` prop은 같다. `document.title`을 매번 카운터 값이 바뀔 때마다 재할당하는 것은 좋아보이지 않는다.

그렇다면, 리액트가 그냥 이펙트를 비교하면 안될까?

```js
let oldEffect = () => { document.title = 'Hello, Dan'; };
let newEffect = () => { document.title = 'Hello, Dan'; };
// 리액트가 이 배열을 같은 배열이라고 인식할 수 있을까?
```

그렇게는 안된다. 리액트는 함수를 호출해보지 않고 함수가 어떤 일을 하는지 알아낼 수 없다.(위 코드는 특정한 값을 포함하고 있는게 아니라 `name `prop에 있는 것을 담아왔을 뿐이다.)

그래서 특정한 이펙트가 불필요하게 다시 실행되는 것을 방지하고 싶다면 의존성 배열을 (`deps` 라고 알려진 녀석) useEffect 의 인자로 전달할 수 있는 것이다.

```js
useEffect(() => {
  document.title = 'Hello, ' + name;
}, [name]); // 우리의 의존성
```

이것은 마치 우리가 리액트에게 '이 함수 안을 볼 수 없다는 것을 알고 있지만, 렌더링 스코프에서 `name` 외의 값은 쓰지 않는다고 약속할게' 라고 말하는 것과 같다.

현재와 이전 이펙트 발동 시 이 값들이 같다면 동기화할 것은 없으니 리액트는 이펙트를 스킵할 수 있다.

```js
const oldEffect = () => { document.title = 'Hello, Dan'; };
const oldDeps = ['Dan'];

const newEffect = () => { document.title = 'Hello, Dan'; };
const newDeps = ['Dan'];

// 리액트는 함수 안을 살펴볼 수 없지만, deps를 비교할 수 있다.
// 모든 deps가 같으므로, 새 이펙트를 실행할 필요가 없다.
```

랜더링 사이에 의존성 배열 안에 있는 값이 하나라도 다르다면 이펙트를 스킵할 수 없다. 모든 것을 동기화 하기 때문에!

<br/>

## 리액트에게 의존성으로 거짓말하지 마라

의존성에 대해 리액트에게 거짓말을 할 경우 좋지 않은 결과를 가져온다. 클래스 컴포넌트에 익숙한 멘탈 모델을 가진 사람들이 `useEffect`를 쓸 때 그 규칙을 어기려는 모습이 보인다.

```jsx
function SearchResults() {
  async function fetchData() {
    // ...
  }
  
  useEffect(() => {
    fetchData();
  }, []); // 이게 맞을까? 항상 그렇진 않다. 그리고 더 나은 방식으로 코드를 작성하는 방법이 있다.
  // ...
}
```

*([Hooks FAQ](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)에서 그 대신 어떻게 해야하는지 설명함. 위의 예제는 아랫부분에서 다시 다룬다.)*

하지만, 마운트 될 때만 이펙트를 실행하고 싶을 때가 있다. 일단 지금은 deps를 지정한다면, **컴포넌트에 있는 모든 값 중 그 이펙트에 사용될 값은 반드시 거기(deps)에 있어야 한다**는 것을 기억하자. props, state, 함수들 등 컴포넌트 안에 있는 모든 것 말이다.

가끔은 위의 코드처럼 하면 문제를 일으킨다. 예를 들어 데이터 불러오는 로직이 무한 루프에 빠질 수도 있고, 소켓이 너무 자주 반응할 수도 있다. **이런 문제를 해결하는 방법은 의존성을 제거하는 것이 `아니다`!**

이 해결책을 알아보기전에, 위의 문제에 대해 더 자세히 알아보자.

<br/>

## 의존성으로 거짓말을 하면 생기는 일

만약 deps가 이펙트에 사용하는 모든 값을 가지고 있다면, 리액트는 언제 다시 이펙트를 실행해야할지 알고 있다.

```js
useEffect(() => {
  document.title = 'Hello, ' + name;
}, [name]);
```

name이 바뀌었을 때만 다시 실행한다는 뜻.

하지만 만약 이펙트에 `[]`를 넘겨주었다면, 새 이팩트 함수는 실행되지 않을 것이다.

```js
useEffect(() => {
  document.title = 'Hello, ' + name;
}, []); // deps에 name이 없다
```

의존성이 같으므로 이펙트는 스킵한다.

이 경우에는 문제가 명확해 보일 수 있다. 하지만 다른 경우에 클래스 스타일의 해결책이 생각나면 이런 직관이 헷갈릴 수 있다.

예를 들어 매 초마다 숫자가 올라가는 카운터를 작성한다고 해보자. 클래스 컴포넌트의 개념을 젹용했을 때 우리는 '인터벌을 한 번만 설정하고, 한 번만 제거하자'가 된다. 그 생각을 코드로 옮기면 [이런 예제](https://codesandbox.io/s/n5mjzjy9kl)가 됨(아래와 같이).

```js
componentDidMount() {
  this.interval = setInterval(this.tick, 1000);
}
componentWillUnmount() {
  clearInterval(this.interval);
}
```

이 멘탈 모델을 가지고 `useEffect` 를 사용한 코드로 변환하게 되면, 직관적으로 deps에 `[]` 를 넣게 됨. "이게 한번만 실행됐으면 좋겠어." 라고.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return <h1>{count}</h1>;
}
```

이렇게. 하지만 이 예제는 숫자가 [오로지 한 번만 증가함](https://codesandbox.io/s/91n5z8jo7r).

만약 우리의 멘탈 모델이 '의존성 배열은 내가 언제 이펙트를 다시 실행해야 할지 지정할 때 쓰인다.' 라면, 위 예제를 볼 때 혼란에 빠질 것임. 이건 인터벌이니까 한 번만 실행하고 싶다. 그런데 왜 문제가 일어났을까?

하지만 의존성 배열이 리액트에게 어떤 랜더링 스코프에서 나온 값 중 이펙트에 쓰이는 것 *전부*를 알려주는 힌트라고 인식한다면 말이 된다. `count`를 사용하지만, deps를 `[]` 라고 정의하면서 거짓말을 했다. 이 거짓말 때문에 버그가 터짐.

첫 번째 랜더링에서 `count`는 `0`이다. 따라서 첫 번째 랜더링의 이펙트에서 `setCount(count + 1)`은 `setCount(0 + 1)` 이라는 뜻이 된다. deps를 `[]`이라고 정의했기 때문에 이펙트를 절대 다시 실행하지 않고, 결국 그로 인해 매 초마다 `setCount(0 + 1)`을 호출하는 것이다.

```jsx
// 첫 번째 랜더링, state는 0
function Counter() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // 언제나 setCount(1)
      }, 1000);
      return () => clearInterval(id);
    },
    [] // 절대 다시 실행하지 않는다
  );
  // ...
}

// 매번 다음 랜더링마다 state는 1이다
function Counter() {
  // ...
  useEffect(
    // 이 이펙트는 언제나 무시될 것
    // 왜냐면 리액트에게 빈 deps를 넘겨주는 거짓말을 했기 때문
    () => {
      const id = setInterval(() => {
        setCount(1 + 1);
      }, 1000);
      return () => clearInterval(id);
    },
    []
  );
  // ...
}
```

우리는 리액트에게 이 이펙트는 컴포넌트 안에 있는 값을 쓰지 않는다고 거짓말을 했다. 실제로는 쓰는데!

이펙트는 컴포넌트 안에 있는 값인(하지만 이펙트 바깥에 있는) `count` 값을 쓰고 있다.

```js
const count = // ...

useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, []);
```

따라서 `[]`을 의존성 배열로 지정하는 것은 버그를 만든다. 리액트는 배열을 비교하고, 이 이펙트를 업데이트 하지 않을 것이다. (의존성이 같으므로 이펙트는 스킵한다)

이런 종류의 이슈는 해결책을 떠올리기 어렵다. 따라서 언제나 이펙트에 의존성을 솔직하게 전부 명시하는 것을 중요한 규칙으로 받아들어야 한다고 권함. lint를 사용한다던지..

<br/>

## 의존성을 솔직하게 적는 두 가지 방법

일반적으로는 첫 번째 방법 사용, 필요하다면 두 번째 방법을 사용하라.

### 1. 컴포넌트 안에 있으면서 이펙트에서 사용되는 모든 값이 의존성 배열 안에 포함되도록 고치는 것

`count`를 deps에 추가해보겠다.

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);   // count 값이 사용되었으니
  }, 1000);
  return () => clearInterval(id);
}, [count]);   // deps에 추가해준다.
```

이렇게 의존성 배열을 올바르게 만들었다. 이상적이지 않을 수 있지만 우리가 고쳐야 하는 첫 번째 문제를 해결한 것. 이제 `count` 값은 이펙트를 다시 실행하고 매번 다음 인터벌에서 `setCount(count + 1)` 부분은 해당 랜더링 시점의 `count` 값을 사용할 것이다.

```jsx
// 첫 번째 랜더링, state는 0
function Counter() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [0] // [count]
  );
  // ...
}

// 두 번째 랜더링, state는 1
function Counter() {
  // ...
  useEffect(
    // 두 번째 랜더링의 이펙트
    () => {
      const id = setInterval(() => {
        setCount(1 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [1] // [count]
  );
  // ...
}
```

이렇게 하면 [문제를 해결하겠지만](https://codesandbox.io/s/0x0mnlyq8l) `count` 값이 바뀔 때마다 인터벌은 해제되고 다시 설정될 것. 원하지 않던 동작이다.

![스크린샷 2021-03-10 오후 3 33 26](https://user-images.githubusercontent.com/59427983/110586756-0140ff80-81b6-11eb-9f9e-54bbe3d39c9e.png)

이렇게.

### 2. 이펙트의 코드를 바꿔 우리가 원하는 것 보다 자주 바뀌는 값을 요구하지 않도록 만드는 것

의존성에 대해 거짓말을 할 필요가 없음. 그냥 의존성을 더 적게 넘겨주도록 바꾸면 된다. 의존성을 제거하는 몇 가지 공통적인 기술을 살펴보자.

<br/>

## 이펙트가 자급자족 하도록 만들기

우리는 이펙트의 의존성에서 `count` 를 제거하도록 만들고 싶다.

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

문제를 해결하기 위해 먼저 자신에게 질문해야 함. **무엇 때문에** count를 쓰고 있나? 오로지 setCount를 위해 사용하고 있는 것으로 보인다. 이 경우 스코프 안에서 count를 쓸 필요는 전혀 없다. 이전 상태를 기준으로 상태 값을 업데이트 하고 싶을 때는, `setState` 에 [함수 형태의 업데이터를](https://reactjs.org/docs/hooks-reference.html#functional-updates) 사용하면 됨. 펑셔널 업데이트는 [여기](https://github.com/pozafly/TIL/blob/main/react/%EA%B8%B0%EB%B3%B8/2.3%20useState%EB%B9%84%EB%8F%99%EA%B8%B0%20%26%20%ED%95%A8%EC%88%98%ED%98%95%20%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8.md)서 정리해 둔게 있긴 하다.

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1);   // 함수형 업데이터
  }, 1000);
  return () => clearInterval(id);
}, []);
```

위 같은 경우를 '잘못된 의존관계' 라고 생각하기 쉽다. 맞음. `count`는 우리가 `setCount(count + 1)` 이라고 썼기 때문에 이펙트 안에서 필요한 의존성이었다. 하지만 우리는 `count`를 `count + 1` 로 변환하여 리액트에게 '돌려주기 위해' 원했을 뿐이다. 하지만 리액트는 현재의 `count`를 이미 알고 있다. **우리가 리액트에게 알려줘야 하는 것은 지금 값이 뭐든간 상태 값을 하나 더하라는 것임**.

그게 정확히 `setCount(c => c + 1)` 이 의도하는 것이다. 리액트에게 상태가 어떻게 바뀌어야 하는지 '지침을 보내는 것'이라고 생각할 수 있다. 이 '업데이터 형태' 또한 다른 케이스에서 사용할 수 있다. 예를 들어 [여러 개의 업데이트를 묶어서 처리할 때처럼.](https://overreacted.io/react-as-a-ui-runtime/#batching)

*의존성을 제거하지 않고도 실제로* 문제를 해결했다는 것을 알아두어야 함. 꼼수 쓰지 않은 것임. 이펙트는 더 이상 랜더링 스코프에서 `count` 값을 읽어 들이지 않는다.

[여기](https://codesandbox.io/s/q3181xz1pj)서 실험해볼 수 있다.

이 이펙트가 한 번만 실행되었다 하더라도, 첫 번째 랜더링에 포함되는 인터벌 콜백은 인터벌이 실행될 때마다 `c => c + 1` 이라는 업데이트 지침을 전달하는데 완벽하게 들어맞는다. 더 이상 현재의 `count` 상태를 알고 있을 필요가 없다. 리액트가 이미 알고 있으니까.

<br/>

## 함수형 업데이트와 구글 닥스(Google Docs)

어떻게 동기화가 이펙트의 멘탈 모델이 되는지 기억남? 동기화에 대해 생각할 때 흥미로운 점은 종종 시스템 간의 `메세지`를 상태와 엮이지 않은 채로 유지하고 싶을 때가 있다는 것임. 예를 들어 구글 닥스에서 문서를 편집하는 것은 실제로 서버 **전체** 페이지를 보내는 것이 아니다. 그렇다면 매우 비효율 적이다. 그 대신 사용자가 무엇을 하고자 했는지 표현한 것을 보낸다.

우리가 사용하는 경우는 다르겠지만 이펙트에도 같은 철학이 적용됨. **오로지 필요한 최소한의 정보를 이팩트 안에서 컴포넌트로 전달하는게 최적화에 도움이 된다.** `setCount(c => c + 1)` 같은 업데이터 형태는 `setCount(count + 1)` 보다 명백히 적은 정보를 전달한다. 현재의 카운트 값에 '오염되지' 않기 때문임. 그저 행위('증가')를 표현할 뿐이다. 리액트로 생각하기 문서에 [최소한의 상태를 찾으라는](https://ko.reactjs.org/docs/thinking-in-react.html#step-3-identify-the-minimal-but-complete-representation-of-ui-state) 내용이 포함되어 있음. 그 문서에 쓰인 것과 같은 원칙이지만 업데이트에 해당된다고 생각하자.

(결과보다) 의도를 인코딩하는 것은 구글 닥스가 협동 편집 문제를 해결한 방법과 유사하다. 과장일 수 있지만 함수형 업데이트 방식도 리액트 안에서 비슷한 역할을 하고 있다. 여러 소스에서 이루어지는 업데이트가 (이벤트 핸들러, 이펙트 구독 등) 예측 가능한 방식으로 모여 정확히 적용될 수 있도록 보장한다.

하지만 `setCount(c => c + 1)`조차도 그렇게 좋은 방법은 **아니다**. 뭐 어쩌라고;; 좀 이상해보이기도 하고 할 수 있는 일이 굉장히 제한적이다. 예를 들어 서로에게 의존하는 두 상태 값이 있거나 prop 기반으로 다음 상태를 계산할 필요가 있을 때는 도움이 되지 않는다. 다행히도 `setCount(c => c + 1)` 은 더 강력한 자매 패턴이 있다. 바로 `useReducer`임.

<br/>

## 액션을 업데이트로부터 분리하기

이전 예제를 `count`와 `step` 두 가지 상태 변수를 가지는 것으로 바꿔보겠다. 인터벌은 `step` 입력값에 따라 `count` 값을 더할 것임.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);   // 여기
    }, 1000);
    return () => clearInterval(id);
  }, [step]);   // 여기
  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />  // 여기
    </>
  );
}
```

(여기 [데모](https://codesandbox.io/s/zxn70rnkx)가 있다.)

이것은 꼼수가 아니다. `step`을 이펙트 안에서 사용하고 있기 때문에 의존성 배열에 추가되었다. 따라서 코드는 정확히 돌아감.

이 예제의 현재 동작은 `step`이 변경되면 인터벌을 다시 시작하는 것임. 왜냐면 의존성으로 정의되어 있으므로. 그리고 많은 경우에 이게 정확히 우리가 원하는 동작일 것이다. 이펙트를 분해하고 새로 설정하는데는 아무 문제가 없고, 특별히 좋은 이유가 있지 않다면 분해하는 것을 피하지 말아야 함.

하지만 `step`이 바뀐다고 인터벌 시계가 초기화되지 않는 것을 원한다면 어떻게 해야하나? 이펙트의 의존성 배열에서 `step`을 제거하려면?

**어떤 상태 변수가 다른 상태 변수의 현재 값에 연관되도록 설정하려고 한다면, 두 상태 변수 모두 useReducer로 교체해야할 수 있다.**

`setSomething(something => …)` 같은 코드를 작성하고 있다면, 대신 리듀서를 써보는 것을 고려하기 좋은 타이밍임. 리듀서는 컴포넌트에서 일어난 '액션'의 표현과 그 반응으로 상태가 어떻게 업데이트 되어야 할지를 분리한다.

이펙트 안에서 `step` 의존성을 `dispatch`로 바꿔 보겠다.

```js
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' });  // setCount(c => c + step) 대신에
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

([데모](https://codesandbox.io/s/xzr480k0np)를 보자.)

아마 "이게 뭐가 더 좋음?" 라고 물어볼 수 있다. **리액트는 컴포넌트가 유지되는 한 `dispatch` 함수가 항상 같다는 것을 보장함.** 따라서 위의 예제에서 인터벌을 다시 구독할 필요조차 없음.

문제가 해결!

*(리액트가 `dispatch`, `setState`, `useRef` 컨테이너 값이 항상 고정되어 있다는 것을 보장하니까 의존성 배열에서 뺄 수도 있다. 하지만 명시한다고 해서 나쁠 것은 없다.)*

이팩트 안에서 상태를 읽는 대신 *무슨 일이 일어났는지* 알려주는 정보를 인코딩 하는 **액션**을 디스패치한다. 이렇게 해서 이팩트는 `step` 상태로부터 분리되어 있다. 이펙트는 *어떻게* 상태를 업데이트 할지 신경쓰지 않고, 단지 **무슨 일이 일어났는지** 알려준다. 그리고 리듀서가 업데이트 로직을 모아둔다.

```js
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

<br/>

## 왜 useReducer가 Hooks의 치트 모드인가

우리는 이팩트가 이전 상태를 기준으로 상태를 설정할 필요가 있을 때 어떻게 의존성을 제거하는지 살펴봤다. **하지만 다음 상태를 계산하는데 props가 필요하다면?** 예를 들어 API가 `<Count step={1} />` 인거다. 당연하지만, 이럴 때 의존성으로 `props.step`을 설정하는 것을 피할 수는 없다.

사실은 피할 수 있다! *리듀서 그 자체*를 컴포넌트 안에 정의하여 props를 읽도록 하면 된다.

```jsx
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);
  
  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;   // 여기
    } else {
      throw new Error();
    }
  }
  
  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);
  
  return <h1>{count}</h1>
}
```

이 패턴은 몇 가지 최적화를 무효화하기 때문에 어디서나 쓰진 말자. 하지만 필요하다면 이렇게 리듀서 안에서 props에 접근할 수 있다. (여기 [데모](https://codesandbox.io/s/7ypm405o8q)가 있음.)

이 경우조처 랜더링 간 `dispatch` 의 동일성은 여전히 보장됨. 그래서 원한다면 이펙트의 의존성 배열에서 빼버릴 수 있음. 이펙트가 재실행되도록 만들지 않을 것이기 때문.

'이게 어떻게 가능하지? 어떻게 다른 랜더링에 포함된 이펙트 안에서 호출된 리듀서가 props를 **알고** 있지?'라고 생각할 수 있다. 답은 `dispatch` 를 할 때에 있다. 리액트는 그저 액션을 기억해 놓는다. 하지만 다음 랜더링 중에 리듀서를 *호출*할 것이다. 이 시점에서 새 props가 스코프 안으로 들어오고 이펙트 내부와는 상관 없게 된다.

따라서 저자는 `useReducer`를 Hooks의 '치트모드' 라고 생각한다. 업데이트 로직과 그로 인해 무엇이 일어나는지 서술하는 것을 분리할 수 있도록 만들어주는 것. 그 다음은 이펙트의 불필요한 의존성을 제거하여 필요할 때보다 더 자주 실행되는 것을 피할 수 있도록 도와준다.

<br/>

## 함수를 이펙트 안으로 옮기기

흔한 실수 중 하나가 함수는 의존성에 포함되면 안된다는 것임. 예를 들어 이 코드는 동작하는 것처럼 보인다.

```jsx
function SearchResults() {
  const [data, setData] = useState({ hits: [] });
  
  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }
  
  useEffect(() => {
    fetchData();
  }, []); // 이거 괜찮은가?
  // ...
```

[예제](https://codesandbox.io/s/8j4ykjyv0).

일단 이 코드는 *동작한다.* 하지만 간단히 로컬 함수를 의존성에서 제외하는 해결책은 컴포넌트가 커지면서 모든 경우를 다루고 있는지 보장하기 아주 힘들다는 문제가 있다.

각 함수가 5배 정도는 커져서 코드를 이런 방식으로 나누었다고 생각해보자.

```jsx
function SearchResults() {
  // 이 함수가 길다고 상상해 보자
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=react';
  }
  
  // 이 함수도 길다고 상상해 보자
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }
  
  useEffect(() => {
    fetchData();
  }, []);
  // ...
}
```

이제 나중에 이 함수들 중 하나가 state나 props를 사용한다고 가정해보자.

```jsx
function SearchResults() {
  const [query, setQuery] = useState('react');
  
  // 이 함수가 길다고 상상해 보자
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }
  
  // 이 함수가 길다고 상상해 보자
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }
  
  useEffect(() => {
    fetchData();
  }, []);
  // ...
}
```

만약 이런 함수를 사용하는 어떠한 이펙트에도 deps를 업데이트 하는 것을 깜빡했다면(아마도 다른 함수를 통해서!), 이펙트는 prop과 state의 변화에 동기화하는데 실패할 것이다. 좋지 않다.

다행히 이 문제를 해결할 쉬운 방법이 있다. 어떠한 함수를 이펙트 *안에서만* 쓴다면, 그 함수를 직접 이펙트 *안으로* 옮기자.

```jsx
function SearchResults() {
  // ...
  useEffect(() => {
    // 아까의 함수들을 안으로 옮겼음!
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }
    
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }
    fetchData();
  }, []); // ✅ Deps는 OK
  // ...
}
```

([여기 데모](https://codesandbox.io/s/04kp3jwwql))

우리는 더이상 '옮겨지는 의존성'에 신경 쓸 필요가 없다. 의존성 배열은 더 이상 거짓말 하지 않는다. **진짜로 이펙트 안에서 컴포넌트의 범위 바깥에 있는 그 어떠한 것도 사용하지 않고 있다.**

나중에 `getFetchUrl`을 수정하고 `query` state를 써야한다고 하면, 이펙트 *안에 있는* 함수만 고치면 된다. 또 `query`를 이팩트의 의존성을 추가해야겠다.

```js
function SearchResults() {
  const [query, setQuery] = useState('react');
  
  useEffect(() => {
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }
    
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }
    
    fetchData();
  }, [query]); // ✅ Deps는 OK
  // ...
}
```

([데모](https://codesandbox.io/s/pwm32zx7z7).)

이 의존성을 더하는 것이 단순히 '리액트를 달래는' 것은 아니다. `query` 가 바뀔 때 데이터를 다시 **페칭**하는 것이 말이 된다. `useEffect`의 디자인은 사용자가 제품을 사용하다 겪을 때까지 무시하는 대신, 데이터 흐름의 변화를 알아차리고 이펙트가 어떻게 동기화해야할지 선택하도록 강제한다.

`eslint-plugin-react-hooks` 플러그인의 `exhaustive-deps` 린트 룰 덕분에, [에디터에서 코드를 작성하면서 이펙트를 분석하고](https://github.com/facebook/react/issues/14920) 어떤 의존성이 빠져 있는지 제안을 받을 수 있게 되었음. 다르게 이야기하자면 컴포넌트 안에서 어떤 데이터 흐름의 변화가 제대로 처리되지 않고 있는지 컴퓨터가 알려줄 수 있다는 것.

![exhaustive-deps](https://user-images.githubusercontent.com/59427983/110596613-d7420a00-81c2-11eb-93b5-526a60144251.gif)

<br/>

## 하지만 저는 이 함수를 이펙트 안에 넣을 수 없어요

때로 함수를 이펙트 안에 옮기고 싶지 않을 수도 있음. 예를 들어 한 컴포넌트에서 여러 개의 이펙트가 있는데 같은 함수를 호출할 때, 로직을 복붙하고 싶지 않다. 아니면 prop 때문이거나.

이런 함수를 이펙트의 의존성으로 정의하지 말아야 하나? 아니다. 다시 말하면 **이펙트는 자신의 의존성에 대해 거짓말을 하면 안된다.** 보통은 더 나은 해결책이 있다. 흔한 오해 중 하나가 '함수는 절대 바뀌지 않는다' 이다. 하지만 이 글을 통해 배웠듯, 그 말이 진실로부터 얼마나 멀리 떨어져있는지 알 수 있다. 사실은, 컴포넌트 안에 정의된 함수는 매 랜더링마다 바뀐다!

하지만 그로 인해 문제가 발생함. 두 이펙트가 `getFetchUrl`을 호출한다고 생각해보자.

```js
function SearchResults() {
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }
  
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl
  
  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl
  // ...
}
```

이 경우 `getFetchUrl`을 각각의 이펙트 안으로 옮기게 되면 로직을 공유할 수 없으니 그다지 달갑지 않다. 두 이펙트 모두(매 랜더링 마다 바뀌는) `getFetchUrl`에 기대고 있으니, 의존성 배열도 쓸모가 없다.

```js
function SearchResults() {
  // 🔴 매번 랜더링마다 모든 이펙트를 다시 실행한다
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }
  
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // 🚧 Deps는 맞지만 너무 자주 바뀐다
  
  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // 🚧 Deps는 맞지만 너무 자주 바뀐다
  // ...
}
```

생각나는 해결책은 그냥 `getFetchUrl`를 의존성 배열에서 빼버리는 것. 하지만 좋은 방법이라 생각되지 않음. 이렇게 하면 우리가 언제 이펙트에 의해 다루어질 *필요가 있는* 데이터 흐름에 변화를 *더해야 할지* 알아차리기 어려워진다.

대신 더 간단한 해결책이 2개 있음.

### 1. 함수가 컴포넌트 스코프 안의 어떠한 것도 사용하지 않는다면, 컴포넌트 외부로 끌어올려두고 이펙트 안에서 자유롭게 사용하면 됨

```js
// ✅ 데이터 흐름에 영향을 받지 않는다
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // ✅ Deps는 OK
  
  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // ✅ Deps는 OK
  // ...
}
```

저 함수는 렌더링 스코프에 포함되어 있지 않으며 데이터 흐름에 영향을 받을 수 없기 때문에 deps에 명시할 필요가 없다. 우연히라도 props나 state를 사용할 수 없음.

## 2. `useCallback` 훅으로 감쌀 수 있음

```js
function SearchResults() {
  // ✅ 여기 정의된 deps가 같다면 항등성을 유지한다
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []);  // ✅ 콜백의 deps는 OK
  
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK
  
  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK
  // ...
}
```

`useCallback` 은 의존성 체크에 레이어를 하나 더 더하는 것임. 다른 말로 문제를 다른 방식으로 해결하는데, 함수의 의존성을 피하기보다 함수 자체가 필요할 때만 바뀔 수 있도록 만드는 것임.

이 접근방식이 왜 유용한지 살펴보자. 아까 예제는 `'react'`, `'redux'` 라는 두 가지 검색 결과를 보여줬음. 하지만 입력을 받는 부분을 추가하여 임의의 `query` 를 검색할 수 있다고 해보자. 그래서 `query` 를 인자로 받는 대신, `getFetchUrl` 이 지역 상태로부터 이를 읽어들임.

그렇게 수정하면 즉시 `query` 의존성이 빠져있다는 사실을 파악하게 됨.

```js
function SearchResults() {
  const [query, setQuery] = useState('react');
  const getFetchUrl = useCallback(() => { // No query argument
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []); // 🔴 빠진 의존성: query
  // ...
}
```

`useCallback` 의 deps에 `query` 를 포함하도록 고치면, `getFetchUrl` 을 사용하는 어떠한 이펙트라도 `query` 가 바뀔 때마다 다시 실행될 것임.

```js
function SearchResults() {
  const [query, setQuery] = useState('react');
  
  // ✅ query가 바뀔 때까지 항등성을 유지한다
  const getFetchUrl = useCallback(() => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]);  // ✅ 콜백 deps는 OK
  
  useEffect(() => {
    const url = getFetchUrl();
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK
  // ...
}
```

`useCallback` 덕분에 `query` 가 같다면, `getFetchUrl` 또한 같을 것이며, 이펙트는 다시 실행되지 않을 것. 하지만 만약 `query` 가 바뀐다면, `getFetchUrl` 또한 바뀌며, 데이터를 다시 페칭할 것임. 마치 엑셀 스프레드시트에서 어떤 셀을 바꾸면 다른 셀이 자동으로 다시 계산되는 것과 비슷하다.

이것은 그저 데이터 흐름과 동기화에 대한 개념을 받아들인 결과다. **부모로부터 함수 prop을 내려보내는 것 또한 같은 해결책이 적용된다.**

```js
function Parent() {
  const [query, setQuery] = useState('react');
  
  // ✅ query가 바뀔 때까지 항등성을 유지한다
  const fetchData = useCallback(() => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + query;
    // ... 데이터를 불러와서 리턴한다 ...
  }, [query]);  // ✅ 콜백 deps는 OK
  return <Child fetchData={fetchData} />
}
  
function Child({ fetchData }) {
  let [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ 이펙트 deps는 OK
  // ...
}
```

`fetchData` 는 오로지 `Parent` 의 `query` 상태가 바뀔 때만 변하기 때문에, `Child` 컴포넌트는 앱에 꼭 필요할 때가 아니라면 데이터를 다시 페칭하지 않을 것이다.

<br/>

## 함수도 데이터 흐름의 일부인가?

흥미롭게 이 패턴은 클래스 컴포넌트에서 사용하면 제대로 동작하지 않는데, 이게 이펙트와 라이프사이클 패러다임의 결정적인 차이를 보여준다. 위 코드를 클래스 컴포넌트로 치환해보자.

```js
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... 데이터를 불러와서 무언가를 한다 ...
  };
  render() {
    return <Child fetchData={this.fetchData} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  render() {
    // ...
  }
}
```

`useEffect` 는 `componentDidMount` 와 `componentDidUpdate` 가 섞인 것이라고 생각할 수 있다. **하지만 이 로직은 `componentDidUpdate` 에선 동작하지 않는다.**

```js
class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    // 🔴 이 조건문은 절대 참이 될 수 없다
    if (this.props.fetchData !== prevProps.fetchData) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

당연히 `fetchData`는 클래스 메서드다! state가 바뀌었다고 저 메서드가 달라지지 않는다. 따라서 `this.props.fetchData`는 `prevProps.fetData`와 같기 때문에 절대 다시 데이터를 페칭하지 않는다. 그러면 아까 조건문을 제거하면 어때?

```js
componentDidUpdate(prevProps) {
  this.props.fetchData();
}
```

이러면 *매번* 다시 랜더링 할 때마다 데이터를 불러온다. 혹시 특정한 `query`를 바인딩 해두면 어떨까?

```jsx
render() {
  return <Child fetchData={this.fetchData.bind(this, this.state.query)} />;
}
```

하지만 이렇게 하면 `query`가 바뀌지 않았는데도 `this.props.fetchData!== prevProps.fetchData` 는 언제나 `true` 가 될 것임! 결국 매번 데이터를 다시 페칭하겠다.

진짜 클래스 컴포넌트로 이 문제를 해결하는 방법은 `query` 자체를 `Child` 컴포넌트에 넘기는 것 뿐이다. `Child` 컴포넌트가 `query` 를 직접 사용하지 않음에도 불구하고 `query` 가 바뀔 때 다시 데이터를 불러오는 로직은 해결할 수 있다.

```jsx
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... 데이터를 불러와서 무언가를 한다 ...
  };
  render() {
    return <Child fetchData={this.fetchData} query={this.state.query} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    if (this.props.query !== prevProps.query) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

저자는 수 년동안 리액트의 클래스 컴포넌트를 다뤄 오면서, 불필요한 prop을 내려보내면서 부모 컴포넌트의 캡슐화를 깨트리는 일에 익숙해졌다고 한다. 그러다 이제사 왜 그래야 했는지 깨달았다고 한다.

**클래스 컴포넌트에서, 함수 prop 자체는 실제로 데이터 흐름에서 차지하는 부분이 없다.** 메소드는 가변성이 있는 `this` 변수에 묶여 있기 때문에 함수의 일관성을 담보할 수 없게 된다. 그러므로 우리가 함수만 필요할 때도 '차이' 를 비교하기 위해 다른 데이터를 전달해야 했다. 부모 컴포넌트로부터 내려온 `this.props.fetchData` 가 어떤 상태에 기대고 있는지, 아니면 그냥 상태가 바뀌기만 한 것인지 알 수가 없다.

**`useCallback` 을 사용하면, 함수는 명백하게 데이터 흐름에 포함된다.** 만약 함수의 입력값이 바뀌면 함수 자체가 바뀌고, 만약 그렇지 않다면 같은 함수로 남아있다고 말 할 수 있다. 세밀하게 제공되는 `useCallback` 덕분에 `props.fetchData` 같은 props 변화는 자동적으로 하위 컴포넌트로 전달된다.

비슷하게, [`useMemo`](https://reactjs.org/docs/hooks-reference.html#usememo) 또한 복잡한 객체에 대해 같은 방식의 해결책을 제공한다.

```js
function ColorPicker() {
  // color가 진짜로 바뀌지 않는 한
  // Child의 얕은 props 비교를 깨트리지 않는다
  const [color, setColor] = useState('pink');
  const style = useMemo(() => ({ color }), [color]);
  return <Child style={style} />;
}
```

**그렇다고 `useCallback` 을 어디든지 사용하는 것은 꽤 투박한 방법이다.** `useCallback` 은 꽤 좋은 해결책이고, 함수가 전달되어 자손 컴포넌트의 이펙트 안에서 호출되는 경우 유용하다. 아니면 자손 컴포넌트의 메모리제이션이 깨지지 않도록 방지할 때도 쓰인다. 하지만 훅 자체가 [콜백을 내려보내는 것을 피하는](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) 더 좋은 방법을 함께 제공한다.

위의 예제라면 `fetchData` 를 이펙트 안에 두거나(커스텀 훅으로 추상화될 수도 있고) 최상위 레벨로 만들어 import 할 것이다. 이펙트를 최대한 단순하게 만들고 싶고 그 안에 콜백을 두는 것은 단순하게 만드는데 별로 도움이 되지 않는다. ('만약에 어떤 `props.onComplete` 콜백이 요청이 보내지고 있을 때 실행된다면?') [클래스 컴포넌트의 동작을 흉내낼순 있지만](https://overreacted.io/a-complete-guide-to-useeffect/#swimming-against-the-tide) 경쟁 상태(race condition)를 해결할 수는 없다.

<br/>

## 경쟁 상태에 대해

클래스로 데이터를 불러오는 전통적인 예제는 이렇게 생겼을 것이다.

```js
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

이 코드는 버그가 있다. 컴포넌트가 업뎃 되는 상황을 다루지 않는다. 그래서 두 번째 클래스 컴포넌트 예제를 찾아보면 이렇게 생겼을 것이다.

```js
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {   // 요부분
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

이게 훨씬 낫다! 하지만 여전히 버그가 있다. 버그가 발생하는 이유는, `순서를 보장할 수 없기 때문`이다. 만약 `{ id: 10 }` 으로 데이터를 요청하고 `{ id: 20 }` 으로 바꾸었다면, `{ id: 20 }` 의 요청이 먼저 시작된다. 그래서 먼저 시작된 요청이 더 늦게 끝나서 잘못된 상태를 덮어씌울 수 있다.

이를 `경쟁 상태` 라고 하고, (보통 비동기 호출의 결과가 돌아올 때까지 기다린다고 여기며) 위에서 아래로 데이터가 흐르면서 `async` / `await` 이 섞여있는 코드에 흔히 나타난다. 여기서 데이터가 흘러간다는 말은, props나 state가 async 함수 안에서 바뀔 수 있다는 이야기임.

이펙트가 마법같이 이 문제를 해결해주지 않는다. 거기다 asycn 함수를 이펙트에 직접적으로 전달하면 경고가 나타난다. 우리가 사용하는 비동기 접근 방식이 취소 기능을 지원한다면 아주 좋다! 그러면 클린업 함수에서 바로 비동기 함수를 취소할 수 있다.

또는 제일 쉽게 boolean 값으로 흐름이 멈춰야 하는 타이밍을 조절할 수 있다.

```js
function Article({ id }) {
  const [article, setArticle] = useState(null);
  
  useEffect(() => {
    let didCancel = false;
    async function fetchData() {
      const article = await API.fetchArticle(id);
      if (!didCancel) {
        setArticle(article);
      }
    }
    
    fetchData();
    
    return () => {
      didCancel = true;
    };
  }, [id]);
  // ...
}
```

[이 링크의 글은](https://www.robinwieruch.de/react-hooks-fetch-data/) 어떻게 에러를 다루고 상태를 불러올지, 또한 이런 로직을 어떻게 커스텀 훅으로 추상화할 수 있는지 소개함. 훅으로 데이터를 페칭하는 방법을 더 자세히 알고 싶다면 꼭 읽어보자.

<br/>

## 진입 장벽을 더 높이기

클래스 컴포넌트의 라이프사이클 개념으로 생각하면, 사이드 이펙트는 랜더링 결과물과 다르게 동작한다. UI를 랜더링하는 것은 props와 state를 통해 이루어지며 이들의 일관성에 따라 보장받는다. 하지만 사이드 이펙트는 아니다. 이게 흔히 버그가 일어나는 원인임.

`useEffect` 의 개념으로 생각하면, 모든 것들은 기본적으로 `동기화` 됨. 사이드 이펙트는 리액트의 데이터 흐름의 일부다. 모든 `useEffect` 호출에서, 제대로만 코드를 작성했다면 엣지 케이스를 훨씬 쉽게 다룰 수 있다.

하지만 제대로 `useEffect` 를 다루기 위한 초기 비용은 높다. 엣지 케이스를 잘 다루는 동기화 코드를 작성하는 일은 랜더링과 별개로 발생하는 일회성 사이드 이펙트를 실행하는 것 보다 필연적으로 어렵다.

`useEffect` 가 대부분의 시간과 함께 해야하는 *도구* 라면 꽤나 걱정스러운 일. 하지만 `useEffect` 는 낮은 수준의 레이어(로우 레벨)에 위치한 구성 요소라고 함….

저자는 서로 다른 앱이 자신만의 `useFetch` 훅을 만들어 인증 로직을 캡슐화하거나, `useTheme` 를 사용하여 테마 컨텍스트를 사용하는 모습을 봤다고 한다. 이런 도구 모음을 가지게 되면, `useEffect` 를 직접 쓸 일은 *그리* 많지 않을 것이다. 하지만 그 유연함은 모든 훅을 만드는데 충분히 유용하게 작용할 수 있다.

아직까지는 `useEffect` 가 가장 흔히 사용되는 경우는 데이터 페칭이다. 하지만 데이터 페칭은 엄밀히 말해 동기화의 문제는 아니다. 특히나 명백한 것이 주로 이럴 때는 deps가 `[]` 가 되기 때문임. 도대체 우리는 무엇을 동기화하고 있을까?

장기적인 관점에서 [데이터 불러오는 로직을 해결하기 위해 Suspense가 도입되면](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#react-16x-mid-2019-the-one-with-suspense-for-data-fetching) 서드 파티 라이브러리들이 기본적으로 리액트에게 어떤 비동기적인 행위(코드, 데이터, 이미지 등 모든 것)가 준비될 때까지 랜더링을 잠시 미룰 수 있도록 지원하게 될 것이다.

Suspense가 자연스럽게 데이터를 불러오는 경우를 더 많이 커버하게 되면, 예측컨대 `useEffect` 는 더욱 로우 레벨로 내려가 파워유저들이 진정으로 사이드 이펙트를 통해 props와 state를 동기화하고자 할 필요가 있을 때 사용하는 도구가 될 것이다. 데이터 페칭과 다르게 원래 디자인 된 대로 이런 동기화 케이스를 자연스럽게 다룬다. 하지만 그 때까지는, [여기 있는 것과 같은](https://www.robinwieruch.de/react-hooks-fetch-data/) 커스텀 훅이 재사용 가능한 데이터 페칭 로직을 다루는 좋은 방법이다.
