# Ref와 멤버변수

함수 외부에서 변수를 선언하면 ref와 동일하지 않을까? 해서 한 실험.

## ref

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

## 멤버변수

```jsx
let ref = 0;
export default function Counter() {
  function handleClick() {
    ref = ref + 1;
    alert('You clicked ' + ref + ' times!');
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

그리고 이를 App에서 아래와 같이 선언함.

```jsx
export default function App() {
  return (
    <>
      <Counter />
      <Counter />
      <Counter />
      <Counter />
      <Counter />
    </>
  );
}
```

그리고 각 버튼을 클릭해본다.

- ref : 각 컴포넌트마다 count가 찍히는 것이 다름. 즉, 독립적이다.
- 멤버변수 : ref 값이 동기화 되어 있음. 어느 버튼을 클릭하던지 계속 누적된 값이 나타남.

즉, 멤버 변수를 사용하면 값을 독립시킬 수 없고 컴포넌트를 재사용했을 때, 동일 변수를 바라보고 있다.