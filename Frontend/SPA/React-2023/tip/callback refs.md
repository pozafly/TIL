# Callback refs

> [출처](https://velog.io/@cnsrn1874/%EB%B2%88%EC%97%AD-callback-refs-%EC%82%AC%EC%9A%A9%EC%9C%BC%EB%A1%9C-useEffect-%EB%B0%A9%EC%A7%80%ED%95%98%EA%B8%B0)

이론적으로 ref는 임의의 값을 저장할 수 있는 변경 가능한 컨테이너지만, DOM 노드에 접근하는데 자주 사용된다.

```jsx
const ref = React.useRef(null)

return <input ref={ref} defaultValue="Hello world" />
```

`ref`는 리액트 내장 기능의 예약된 속성으로, 렌더링 이후 해당 DOM 노드를 저장한다. 그리고 컴포넌트가 다시 언마운트 되면 다시 null로 설정된다.

## Refs와 상호작용

상호작용하는 대부분의 경우, 리액트가 업데이트를 자동으로 처리하기 때문에 개발자가 DOM 노드에 접근할 필요가 없다. ref를 필요로 하는 좋은 예는 focus 관리다.

## Effect를 이용한 focus

렌더링 이후에 input에 focus하려면 어떻게 해야하나?

```jsx
const ref = React.useRef(null)

React.useEffect(() => {
  ref.current?.focus()
}, []);

return <input ref={ref} defaultValue="Hello world" />
```

위 코드는 대부분 괜찮다. 규칙을 어기지도 않는다. useEffect 내부에서 사용되는 것은 불변하는 ref 뿐이라 의존성 배열이 비어있는 것도 괜찮다. linter는 ref를 의존성 배열에 추가하라고 불평하지 않고, ref는 렌더링 도중에 읽히지 않을 것이다. (concurrent한 리액트의 기능에선 문제가 될 수도 있다.)

effect를 마운트 시 한번 실행된다. 그 때쯤이면 리액트는 이미 ref에 DOM 노드를 추가했을 것이므로 focus할 수 있다.

하지만, 최선의 방법이 아니고, 일부 상황에서는 몇 가지 주의 사항이 있다.

위 코드에서 effect가 실행될 때, ref에는 이미 값이 있는 것으로 가정된다. 하지만, 예를 들어, 개발자가 ref를 커스텀 컴포넌트에 넘겨 렌더링이 지연되는 경우나, 사용자가 다른 상호작용을 마친 뒤 input을 보여주는 경우와 같이 ref에 노드 값이 들어가지 않았을 경우에, effect가 실행될 때의 ref는 여전히 null일 것이다.

> 🚨 주의점은, useEffect는 paint 이후에 실행되는 hook이고, useRef에는 commit phase 이후에 DOM 값이 들어오도록 되어 있다.

```jsx
function App() {
  const ref = React.useRef(null)

  React.useEffect(() => {
    // 🚨 이게 실행될 때, ref.current는 항상 null입니다.
    ref.current?.focus()
  }, [])

  return <Form ref={ref} />
}

const Form = React.forwardRef((props, ref) => {
  const [show, setShow] = React.useState(false)

  return (
    <form>
      <button type="button" onClick={() => setShow(true)}>
        show
      </button>
      // 🧐 ref가 input에 붙어있지만, 조건부 렌더링 됩니다.
      // 그러므로 위의 effect가 실행될 때, ref는 비어있을 겁니다.
      {show && <input ref={ref} />}
    </form>
  )
})
```

실제로 원하는 것은, '**form이 마운트 되었을 때**'가 아니라 '**input이 렌더링 되었을 때**' input에 focus 하는 것인데, 문제는 effect가 Form의 render 함수에 **바인드 되어있다**는 것이다.

## Callback refs

이 때 바로 콜백 ref를 사용할 수 있다. ref 객체 뿐 아니라 함수도 전달할 수 있다.

```js
type Ref<T> = RefCallback<T> | RefObject<T> | null
```

개념적으로, 필자는 **리액트 요소의 ref를 컴포넌트가 렌더링 된 이후에 호출되는 함수**라고 생각하길 좋아한다. 이 함수는 렌더링 된 DOM 노드를 인자로 전달한다. 리액트 요소가 언마운트 되면, 이 함수는 한 번 더 호출되어 null을 전달한다.

그러니까 `useRef`(RefObject)로 부터 리액트 요소에게 ref를 전달하는 것은 단순히 아래 코드의 문법적 설탕이다.(syntatic sugar)

```jsx
<input
  ref={(node) => {
    ref.current = node;
  }}
  defaultValue="Hello world"
/>
```

다시 한번 강조하면 **모든 ref props는 그냥 함수다!**

그리고 이 함수는 사이드 이펙트가 실행되어도 완전히 괜찮은 시기인 렌더링 이후에 실행된다. `ref`가 그냥 `onAfterRender`에 호출되었으면 더 나았을 것이다. 개발자가 직접 노드에 접근할 수 있다는 걸 알게 되었는데, 콜백 ref 내부에서 input에 바로 focus하는 것을 주가 막을 수 있을까?

```jsx
<input
  ref={(node) => {
    node?.focus()
  }}
  defaultValue="Hello world"
/>
```

사소한 디테일이 막을 수 있다. 리액트는 렌더링 할 때마다 매번 이 함수를 실행할 것이다. 그러니 input을 그 정도로 자주 focus하길 원하는 것이 아니라면 (원하지 않을 가능성이 높다), 개발자는 리액트에게 우리가 원할 때만 실행하라고 말해줄 수 있다.

### useCallback

리액트는 콜백 ref가 실행되어야 하는지 확인하기 위해 참조 불변성을 사용한다. 동일한 reference를 전달하면 실행이 생략된다는 뜻이다.

그리고 이 때 함수가 불필요하게 생성되지 않도록 보장하는 방법인 useCallback을 사용할 수 있다. 아마도 이것이 callback-refs라는 이름의 이유일지도 모른다. 왜냐하면 callback refs를 항상 useCallback으로 감싸야 하기 때문이다.

```jsx
const ref = React.useCallback((node) => {
  node?.focus()
}, [])

return <input ref={ref} defaultValue="Hello world" />
```

위 코드를 처믕 버전이랑 비교해보면, 코드가 적고 훅을 하나만 사용한다. 그리고 콜백 ref는 DOM 노드를 마운트 시키는 **컴포넌트의 생명주기가 아니라, DOM 노드의 생명주기에 바인드** 되어 있기 때문에 모든 상황에서 작동할 것이다. 게다가 엄격 모드에서 두 번 실행되지도 않을 것이다.

그리고 모든 종류의 사이드 이펙트를 실행할 때 이것을 사용할 수 있다. (ex. 콜백 ref 내부에서 setState를 호출)

```jsx
function MeasureExample() {
  const [height, setHeight] = React.useState(0)

  const measuredRef = React.useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height)
    }
  }, [])

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  )
}
```

그러면, 렌더링 이후에 DOM 노드와 직접 상호작용 해야한다면, 성급하게 `useRef + useEffect`를 사용하기 보다, 콜백 ref 사용을 고려하자.
