# 제어컴포넌트vs비제어컴포넌트

> - https://goshacmd.com/controlled-vs-uncontrolled-inputs-react/
> - https://www.youtube.com/watch?v=LD1LyvCCbCg

## Input Element

1. input Element는 사용자 입력 값으로 value attribute가 변한다.
2. 사용자가 입력한 값이 value attribute에 저장된다.
3. input 태그의 value attribute는 DOM에 존재한다.
4. input을 통한 사용자 입력 데이터는 DOM에 저장된다.

일반적으로 input을 다룰때, `setState`를 통해 값을 update 하는 것과, `ref`를 사용해 다루는 두가지 방법이 있다. 올바른 방법이란 뭘까?

<br/>

## 비제어컴포넌트와 제어컴포넌트

### 비제어컴포넌트

```jsx
import { useRef, useState } from 'react';

export default function App() {
  const [value, setValue] = useState('');
  const inputRef = useRef(null);

  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => setValue(inputRef.current.value)}>Click</button>
      <span>{value}</span>
    </>
  );
}
```

- 입력한 내용을 기억한다.
- ref를 사용해 값을 얻는다.
- 버튼을 클릭하면 setValue로 상태 값을 업데이트 한다.

즉, 필요할 때 필드에서 값을 **받는(pull)**다. 이는 form 태그를 사용한다면 더 유효한 동작일 수 있다.

> 📌 ref란?
>
> React에서 값을 담는 상자. 컴포넌트 마운트 시점에 current에 element를 대입한다. 컴포넌트가 리렌더링 되어도 ref 값은 유지한다. 컴포넌트의 전역 변수라 볼 수 있다.

<br/>

### 제어 컴포넌트

제어 컴포넌트란, React에 의해 값이 제어되는 입력 폼 엘리먼트를 '제어 컴포넌트' 라고 한다. React state를 '신뢰 가능한 단일 출처(single source of truth)'로 만들어 두 요소를 결합한다.

```jsx
import { useState } from 'react';

export default function App() {
  const [value, setValue] = useState('');

  return (
    <>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <span>{value}</span>
    </>
  );
}

```

- state가 있고, onChange 이벤트가 발생할 때마다 setState를 해주고 있다.
- Form 요소는 명시적으로 요청할 필요 없이 항상 값을 가진다.

입력 할 때마다 값을 **넣는(push)**다. 즉, 데이터(상태)와 UI(입력)가 항상 동기화 된다. 상태는 input의 값을 제공하고, input은 현재 값을 변경하도록 요청한다.

> 📌 신뢰 가능한 단일 출처(single source of truth)
>
> **하나의 상태를 나타내는 state는 한 곳에만 존재해야 한다.**
>
> 어떤 state가 여러 컴포넌트에서 사용 되어야 한다면 props, context api, redux 등으로 관리한다. 하나의 상태는 한 곳에만 존재해야 한다는 뜻이다.
>
> ```js
> let name = '';
> document.querySelector('input').addEventListener('input', (e) => {
>     name = e.target.value;
> });
> ```
>
> 하지만, 이렇게 코드를 사용한다면 값은 2개의 출처가 되는 것이다.
>
> - `name` 변수
> - input DOM의 value attribute
>
> 코드를 작성하기 전까지는 value attribute에만 있었다. 양분화 되면서 어떤 값을 사용할 경우 참조할 수 있을지 헷갈린다. 확실하지 않아졌다.
>
> React가 이 값을 보장해준다. 제어 컴포넌트로. 제어 컴포넌트를 통해 2개의 출처가 된 값이 항상 일치함을 보장한다. 데이터가 불일치 할 염려 없이 사용할 수 있게.

제어 컴포넌트는 아래 상황에서 즉시 응답할 수 있다.

- 유효성 검사와 같은 즉각적인 피드백
- 모든 필드에 유효한 데이터가 없다면 버튼 비활성화
- 신용 카드 번호와 같은 특정 입력 형식 적용

하지만 위의 것들이 실시간으로 평가될 필요가 없다면 비제어 컴포넌트가 더 낫다.

---

제어 컴포넌트는 입력할 때마다 컴포넌트 렌더링이 일어난다. 하지만 비제어 컴포넌트는 사용자가 직접 어떤 행위를 트리거 하기 전에는 리렌더링을 발생시키지 않고, 값도 동기화 하지 않는다.

>왜 비제어 컴포넌트를 사용할 때는 useRef를 사용하고, useRef는 왜 리렌더링을 발생시키지 않나?
>
>`useRef` 는 heap 영역에 저장되는 일반적인 JavaScript 객체다. 매번 렌더링 할 때 동일 객체를 제공한다. heap에 저장되어 있기 때문에 어플리케이션이 종료되거나 가비지 컬렉팅 될 때까지, 참조할 때마다 같은 메모리 값을 가진다고 할 수 있다.
>
>따라서, 값이 변경되어도 리렌더링 되지 않는다. 같은 메모리 주소를 갖고 있기 때문에 자바스크립트의 `===` 동치 연산이 항상 true를 반환함. 즉 변경 사항을 감지할 수 없기 때문에 리랜더링이 발생하지 않음.

---

제어 컴포넌트에서 api 호출이 입력할 때마다 일어나는 경우, 쓰로틀 및 디바운스를 걸어주어 이를 방지할 수 있다.

---

<br/>

이전 공식문서와 업데이트된 공식문서에서 약간 뉘앙스를 다르게 말하는 것을 알게 되었음.

📌 제어 컴포넌트와 비제어 컴포넌트

- [이전 버전](https://ko.legacy.reactjs.org/docs/forms.html#controlled-components)
- [업데이트 버전](https://react-ko.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components)

이를 설명하는 것은 React 공식 문서가 업데이트 되면서 2가지 개념이 되었다. 이전 버전에서는 주로 Form을 초점에 두고 설명하고, 업데이트 버전에서는 일반적인 개념 정도로 설명한다.

- Form의 제어, 비제어 컴포넌트
  - 제어 : state를 갖고 있으며, onChange 핸들러를 통해 상태값을 실시간 업데이트 함.
  - 비제어 : ref를 통해 DOM의 값을 가져온다.
- 일반적으로 사용하는 제어, 비제어 컴포넌트
  - 제어 : state를 가지지 않고, 부모 컴포넌트가 props로 제어할 수 있는 컴포넌트.
  - 비제어 : state를 가지며, 부모 컴포넌트에서 제어되지 않는 컴포넌트.

📌 단일 진실 공급원(SSOT)

- [이전 버전](https://ko.legacy.reactjs.org/docs/forms.html#controlled-components)
- [업데이트 버전](https://react-ko.dev/learn/sharing-state-between-components#a-single-source-of-truth-for-each-state)

이도 마찬가지로 약간 뉘앙스가 다르게 표현되었다.

- 이전 : 문맥상 한 곳의 state에서 정보를 관리한다.
- 업데이트 : 모든 state가 한 곳에 있는 것이 아니라, 각 state마다 정보를 소유하는 특정 컴포넌트가 있다.
