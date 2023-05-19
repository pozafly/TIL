# Strict Mode

> https://react-ko.dev/learn/keeping-components-pure#detecting-impure-calculations-with-strict-mode

React에서 렌더링 하는 동안 읽을 수 있는 값이 3개 있다.

- state
- props
- context

이러한 값은 항상 읽기 전용으로 취급해야 한다.

사용자 입력에 대한 응답으로 무언가를 **변경**하려면 변수에 쓰는 대신 state를 설정해야 한다. 컴포넌트가 렌더링 되는 동안에는 기존 변수나 객체를 절대 변경해서는 안된다.

```jsx
import { useState } from 'react';

export default function App() {
  const [value, setValue] = useState(0);
  setValue(1);
  return (
    <>
      <div />
    </>
  );
}
```

이렇게 되면 안된다.

React는 개발 환경에서 각 컴포넌트의 함수를 두 번 호출하는 'Strict Mode'를 제공한다. Strict Mode는 컴포넌트 함수를 두 번 호출함으로써 이러한 규칙을 위반하는 컴포넌트를 찾아내는 데 도움이 된다.

이는 순수한 컴포넌트(순수 함수)인지 체크하기 위함인데, 순수 함수라면 두번 호출해도 아무것도 바뀌지 않기 때문이다.

Strict Mode는 상용 환경에서는 아무런 영향을 미치지 않기 때문에 사용자의 앱 속도가 느려지지 않는다.

루트 컴포넌트를 `<React.StrictMode>`로 감싸면 된다.