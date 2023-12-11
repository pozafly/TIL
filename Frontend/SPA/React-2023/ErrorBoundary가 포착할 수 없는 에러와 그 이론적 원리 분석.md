# ErrorBoundary가 포착할 수 없는 에러와 그 이론적 원리 분석

> [출처](https://happysisyphe.tistory.com/66)

ErrorBoundary는 모든 에러를 잡아주는 것이 아니다. JavaScript와 React를 잘 이해하고 있다면 당연한 현상이다.

![image](https://github.com/pozafly/TIL/assets/59427983/47cef267-bfcc-4b5d-b754-5fc3d6cd2ae5)

<br/>

## [사전지식] 실행 컨텍스트와 예외처리

```javascript
try {
  function throwErrorFn() {
    throw new Error("will it be catched?");
  }
  setTimeout(throwErrorFn, 1000);
} catch (e) {
  console.log(e);
}
```

setTimeout에서 Error를 throw 하고, 저렇게 try / catch를 감싸면 과연 catch 문에 잡힐까?

정답은 아니다. **실행 컨텍스트 때문**이다. 

실행 컨텍스트란, 소스코드 평가 및 실행을 통해 발생하는 환경을 관리하는 객체를 말한다. 쉽게 말하면, 함수 실행에 필요한 변수(Context)를 관리하는 역할을 한다.

```javascript
function foo(a) {
  const x = 10;
  const y = 20;

  function bar() {
    return x + y + a;
  }

  return bar();
}

foo(100);
```

foo 코드가 실행되면, foo 실행컨텍스트가 만들어지고, 실행 컨텍스트에는 내부 변수인 x,y,a,bar 라는 변수를 관리한다.

![image](https://github.com/pozafly/TIL/assets/59427983/b1f91fec-2d26-4387-8f7c-a930f49a5cd5)

추가로 실행 컨텍스트는 **call stack** 이라고 하는 곳에 순차적으로 쌓이고 pop된다. foo 위에 bar가 쌓이고, bar가 실행되고 나면, bar가 pop되고 foo만 남게 된다. foo까지 실행되고 나면 foo도 pop되고, 콜스택은 비게 된다.

![image](https://github.com/pozafly/TIL/assets/59427983/6233c2f3-cdac-44bf-b339-76ea3539a780)

이때 하나의 context에서 에러가 발생되면 **에러는 상위 컨텍스트로 전파**된다. bar에서 에러가 발생한다하면, bar에서 에러가 throw 되는데, bar에서 처리되지 않으면 foo 까지로 전파된다. foo에서도 처리되지 않으면, **최상위 실행 컨텍스트로 전파되고 crash**가 일어난다.

![image](https://github.com/pozafly/TIL/assets/59427983/54dfc85a-73b0-47eb-a2e7-10b838901abf)

이 배경지식을 기반으로 setTimeout 예제를 다시 보자.

```javascript
try {
  function throwErrorFn() {
    throw new Error("will it be catched?");
  }
  setTimeout(throwErrorFn, 1000);
} catch (e) {
  console.log(e);
}
```

setTimeout 내 callback은 1초 후 실행 컨텍스트에서 실행된다. 이미 try / catch 문이 있던 컨텍스트가 pop된 이후의 상황이기 때문에 catch가 잡을 수 없고, 최상단에 에러가 던져지게 된다. (main 함수는 최상위 컨텍스트 함수를 의미한다.)

![image](https://github.com/pozafly/TIL/assets/59427983/ab694899-edc3-4893-be6a-9b61dded10b3)

즉, `try / catch 문 컨텍스트 내 throwErrrorFn이 싱행되지 않기 때문에 에러를 포착할 수 없`다.

에러 바운더리고 try / catch 와 동일한 원리로 동작한다.

> 에러 경계는 자바스크립트의 catch {} 구문과 유사하게 동작하지만 컴포넌트에 적용됩니다.
>
> -react 공식 문서

에러 바운더리가 잡을 수 있는 에러인지 판단할 때 가장 중요한 것은 '`ErrorBoundary 실행 컨텍스트`' 이다. 그 안에 있다면 잡을 수 있고, 그 안에 없다면 잡을 수 없다. 유일한 원리다.

<br/>

## Case1. 이벤트 핸들러

이벤트 핸들러 내 발생 에러는 ErrorBoundary에 잡히지 않는다.

```jsx
export default function App() {
  return (
    <ErrorBoundary>
      <Button />
    </ErrorBoundary>
  );
}

function Button() {
  return (
    <button
      onClick={() => {
        throw new Error('will it be catched?');
      }}
    >
      click
    </button>
  );
}
```

React의 이벤트 핸들러 바인딩 과정을 이해할 필요가 있음. React 17의 Relase note에 React 이벤트 위임 방식 변화를 알 수 있다. [React v17.0 Release Candidate: No New Features – React Blog](https://ko.legacy.reactjs.org/blog/2020/08/10/react-v17-rc.html#changes-to-event-delegation)

JavaScript에서는 DOM node에 직접 이벤트를 등록한다.

```javascript
document.getElementById("home_button").addEventListener("click", (e) => {
  // callback
});
```

하지만 React에서 모든 이벤트가 root element에서 핸들링한다.

![image](https://github.com/pozafly/TIL/assets/59427983/9e00fa04-0f6b-43a7-9e49-0f6b593e04dc)

아래초럼 console을 찍어보면 root에서 이벤트가 다뤄지고 있음을 알 수 있다.

```jsx
<button
  onClick={(e) => {
    console.log(e.nativeEvent.currentTarget);
  }}
>
  click
</button>
```

<img width="146" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/a442ed7a-31c5-42c7-b2ad-a4b409f302f3">

또한 root에 등록된 eventListeners들을 한 번에 조회해볼 수 있다.

```javascript
getEventListeners(document.getElementById('root'))
```

![image](https://github.com/pozafly/TIL/assets/59427983/1a818207-ec9f-4ea3-a61e-90187c778d9f)

이처럼 모든 이벤트가 사전에 root에 등록되어 있는 것을 볼 수 있다.

React가 이벤트를 핸들링 하는 방식을 알아봤다. 이를 기반으로 왜 ErrorBoundary가 이벤트 핸들러에서 발생한 에러를 잡을 수 없는지 이해할 수 있다.

root는 최상위 tag이고 이곳에서 이벤트 핸들링이 이루어진다. Button 컴포넌트에서 onClick 이벤트가 다루어지는 것처럼 보이지만, 이는 root tag에서 다루어지고 있고, 그곳에서 에러가 발생한다면 그것의 **실행 컨텍스트는, ErrorBoundary 내부에 있지 않다.**

그렇기 때문에 ErrorBoundary가 이벤트 핸들러에서 발생한 에러를 잡을 수 없다. 이벤트 핸들러에서 에러를 핸들링하고 싶다면 이벤트 핸들러 내에서 try / catch 구문을 사용해야 한다.

<br/>

## 비동기적 코드

```jsx
export default function App() {
  return (
    <ErrorBoundary>
      <Children />
    </ErrorBoundary>
  );
}

function Children() {
  function throwErrorFn() {
    throw new Error("will it be catched?");
  }
  setTimeout(throwErrorFn, 1000);
  return <div></div>;
}
```

위에서 확인한 try / catch 예시와 완전히 동일 케이스다.

setTimeout의 callback은 1초 후, 실행 컨텍슽트에 들어와 실행되고, 이는 ErrorBoundary의 컨텍스트가 끝난 시점이다.

![image](https://github.com/pozafly/TIL/assets/59427983/c30b2d76-a40f-4ff0-81fc-fb05b80703f5)

axios 비동기 통신을 보자.

```jsx
export default function App() {
  return (
    <ErrorBoundary>
      <Children />
    </ErrorBoundary>
  );
}

function Children() {
  const [todos, setTodos] = useState([]);
  useEffect(() => {
    (async () => {
      try {
        const res = await axios.get(
          "https://jsonplaceholder.typicode.com/todos21231"
        );
        setTodos(res);
      } catch (e) {
        throw new Error("is it ?");
      }
    })();
  });

  return <button>click</button>;
}
```

여기서도 에러가 잡히지 않는다. ErrorBoudnary가 잡을 수 있도록 하려면 어떻게 해야 하나?

위에서 도출한 원리를 그대로 활용하자면, **ErrorBoundary 내 컨텍스트 내에서 throw를 일으켜야 한다.** 그러므로 error 상태를 별도로 분리, 에러가 있으면 실행 컨텍스트 내 동기적으로 직접 던져버리면 된다.

```jsx
function Children() {
  const [todos, setTodos] = useState([]);
  const [error, setError] = useState(null);

  useEffect(() => {
    (async () => {
      try {
        const res = await axios.get(
          "https://jsonplaceholder.typicode.com/todos21231"
        );
        setTodos(res.data);
      } catch (e) {
        setError(e);
      }
    })();
  }, []);

  if (error) {
    throw error; // 에러 상태가 있으면 에러를 던진다.
  }

  return click;
}
```

react-query 비동기 통신 라이브러리에서 useErrorBoundary(v5에서는 thorwOnError)라는 옵션을 true로 설정해주면 위 기능을 대신해준다.

<br/>

## Case3. SSR

ErrorBoundary는 `getDerivedStateFromError` 를 기반으로, 프론트엔드 상태(hasError)를 변경해 동작한다. 클라이언트 사이드에서만 실행된다. SSR은 서버 사이드(node)에서 API를 요청하고 응답으로 HTML을 만들어 내려주는 기술이다.

getDerivedStateFromError는 상태 변화가 존재하는 브라우저 환경에서만 실행되고, node 환경에서 실행되지 않는다. 따라서 본직적으로 서버 사이드 에러를 포착할 수 없다.

<br/>

## case4. 자식에서가 아닌 에러 경계 자체에서 발생하는 에러

```javascript
try {
  throw new Error("Error");
} catch (e) {
  throw e;
}
```

try에서 에러 발생, catch에서 잡혔는데 이것을 다시 던진다면 catch에서 잡을 수 없다. 이것을 잡기 위한 다른 try/catch가 필요하다.

```javascript
try {
  try {
    throw new Error("Error");
  } catch (e) {
    throw e;
  }
} catch (e) {
  console.log(e);
}
```

ErrorBoundary에서 다시 error가 throw 된다면 에러를 다시 상위 컴포넌트로 던지는 것이다. 상위 ErrorBoundary가 있어야만 포착될 것이고, 그렇지 않다면 최상위에 Error가 도달할 것이다.

<br/>

ErrorBoundary는 부분별로 에러 전파를 방지할 수 있기 때문에 하나의 에러가 어플 전체에 영향을 주는 일을 방지해준다.

