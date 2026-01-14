# Form 개발하기

> [출처](https://oliveyoung.tech/blog/2023-09-18/address-modal/)

form 영역을 [React Hook Form](https://react-hook-form.com/) 이라는 라이브러리를 사용함.

```jsx
const Form = () => {
  return (
    <form>
      <FormTextField label={'배송지 명'} />
      <FormTextField label={'받으실 분'} />
      <FormTextField label={'휴대폰 번호'} />
      <AddressField />
      <EntranceRadio />
      <label>
        <input type="checkbox" /> 공동현관 출입방법 저장 동의
      </label>
      <button>확인</button>
    </form>
  );
};
```

<br/>

## [실험 1] 각 영역의 데이터를 상태로 관리

입력 받고자 하는 데이터를 `<Form>` 컴포넌트 **내부의 상태**로 정의한다.

```jsx
/* <Form> 컴포넌트 */
import { useState } from 'react';
import FormTextField from './FormTextField';

const FormWithState = () => {
  const [addressName, setAddressName] = useState(''); // 배송지 명
  const [receiver, setReceiver] = useState(''); // 받으실 분

  return (
    <form>
      <FormTextField
        label={'배송지 명'}
        onChange={(e) => {
          setAddressName(e.target.value); // 입력이 들어올 때마다 상태 업데이트
        }}
      />
      ..............
    </form>
  );
};
```

form의 onSubmit에서는 데이터 유효성 체크 로직도 추가해보고 최종적으로 데이터도 잘 가져오고 있는지 확인한다.

```jsx
<form
  onSubmit={(e) => {
    e.preventDefault(); // form의 기본 동작으로 페이지가 넘어가는 것을 방지함
    if (addressName === '') {
      alert('배송지를 입력해주세요');
      return;
    }

    const data = {
      addressName,
      receiver,
    };
    console.log('등록하기 ', data);
  }}
>{...}</form>
```

이렇게 해도 form 제출가지 별 문제는 없지만, React Dev Tools를 활용해 성능을 보면 문제가 있다.

![[assets/images/5565417cb7d1b01c7576309fb44b5ccb_MD5.png]]

전체적으로 리랜더링이 발생하고 있다. 나머지 영역의 상태 관리까지 추가되면 복잡도가 많이 올라갈 것임.

<br/>

## [실험 2] Context API

`useState` 로 데이터 상태를 정의하고 입력이 들어올 때마다 상태를 업뎃 해주기위해 change 이벤트 핸들러를 달아주었다. 만약 입력 필드의 컴포넌트가 복잡하고 깊어진다면 계속 props로 전달해주어야 한다. 이런 불편을 해소하기 위해 Context를 사용해보자.

Context로 관리할 최상위인 `<Form>` 컴포넌트에서 정의했고 컴포넌트를 Provider로 감싸준다. Provider로 감싸진 자식 컴포넌트에서 props가 아닌 Context를 통해 공통의 값을 가져와 사용할 수 있다.

```jsx
/* <Form> 컴포넌트 */
import { useState } from 'react';
import FormTextFieldContext from '@/components/Context/FormTextFieldContext';
import { AddressContext } from '@/context/AddressContext';

const FormWithContext = () => {
  const [addressData, setAddressData] = useState({
    addressName: '',
    receiver: '',
    phoneNumber: '',
  });

  return (
    <AddressContext.Provider
      value={{
        value: addressData,
        setValue: (id, value) => {
          setAddressData((prev) => ({ ...prev, [id]: value }));
        },
      }}
    >
      <form
        onSubmit={(e) => {
          e.preventDefault();
          console.log('등록하기 ', addressData);
        }}
      >
        <FormTextFieldContext label={'배송지 명'} id="addressName" />
        ..............
      </form>
    </AddressContext.Provider>
  );
};
```

```jsx
/* 입력 필드 컴포넌트 */
const FormTextFieldContext = ({ label, id }) => {
  const addressContext = useContext(AddressContext); // context 가져옴
  return (
    <label>
      {label}
      <input
        type="text"
        onChange={(e) => {
          const text = e.target.value;
          addressContext.setValue(id, text); // context의 공통 set 함수를 사용
        }}
      />
    </label>
  );
};
```

이렇게 context로 부터 change 이벤트 핸들러를 가져와 최상위에 위치한 `<Form>` 컴포넌트의 데이터를 업뎃 해준다.

실험 1에 비해 props로 직접 전달해주지 않아도 된다는 점은 좋아졌지만, `<Form>` 컴포넌트 내부에서 여전히 상태로 관리하고 있기 때문에 입력이 들어올 때마다 직접 업데이트도 해주어야 하는 것은 물론 실험 1번과 마찬가지로 **리렌더링이 발생**한다.

<br/>

## [실험 3] useRef

1, 2의 실험에서는 `<Form>` 컴포넌트 내부에서 전체 필드의 데이터를 상태로 유지하며 입력받을 때마다 상태를 업뎃 해주었다. 이에 따라 입력받을 때마다 화면이 다시 그려지는 아쉬움이 있었는데, 불필요한 렌더링을 줄일 수 있을까?

React 공식 문서에는 상태가 변해 렌더링이 필요 없는 상황에서는 ref를 사용하도록 제안하고 있고, ref로 DOM 원소 자체도 직접 가리킬 수 있기 때문에 이런 form을 만들 때 적합할 것이다.

```jsx
/* <Form> 컴포넌트 */
import { useRef } from 'react';
import FormTextFieldRef from '@/components/Ref/FormTextFieldRef';

const FormWithRef = () => {
  const addressNameRef = useRef('');

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        const addressName = addressNameRef.current.value;

        if (addressName === '') {
          addressNameRef.current.focus(); // 문제가 되는 부분에 포커스를 줄 수 있음
          return;
        }

        const data = { addressName };
        console.log('등록하기 ', data);
      }}
    >
      <FormTextFieldRef label={'배송지 명'} ref={addressNameRef} /> // ref를 그대로 넘겨줌 ..............
    </form>
  );
};
```

```jsx
const FormTextFieldRef = forwardRef(({ label }, ref) => {
  return (
    <label>
      {label}
      {/* ref를 그대로 넘겨줌 */}
      <input ref={ref} type="text" />
    </label>
  );
});
```

최상단 `<Form>` 컴포넌트에서 ref를 정의하고 실제 입력 필드의 input 까지 그대로 넘겨주면서 직접 DOM 원소에 접근해 입력 값을 가져올 수 있다. 실험 1,2의 change 이벤트 핸들링을 해주지 않아도 되고 무엇보다 입력할 때마다 상태가 변하지 않기 때문에 리렌더링이 발생하지 않는다.

ref를 사용함으로 서 focus를 직접 주는 것도 가능해지며 꼭 입력해야 하는 필드인데 값이 없거나 형식에 맞지 않은 입력이 들어온 영역으로 포커스를 줌으로 사용하에게 알림을 줄 수 있다. State, Context로 관리하는 상황에서도 이 기능을 위해 ref를 사용해야 했을 것이다.

이렇게 보면 ref를 사용하는 것은 괜찮은 선택 같다. 하지만, form 입력 갯수가 많아지고 컴포넌트가 복잡해지면, ref 정의, 데이터 유효성 체크 등을 챙겨야 하는데 개발 복잡도가 올라가게 될 것 같다.

<br/>

## [실험 4] React Hook Form 사용해보기

사용법은 register라는 함수로 입력 영역을 key 값으로 등록하고 해당 영역의 입력 규칙들도 함께 등록할 수 있다. 등록하고 나면 각 영역의 데이터를 개발자가 직접 관리하지 않아도 되고 유효성 체크도 간편하게 된다.

```jsx
/* <Form> 컴포넌트 */
import { useForm } from 'react-hook-form';
import FormTextFieldHookForm from '@/components/HookForm/FormTextFieldHookForm';

const FormWithHookForm = () => {
  const { register, handleSubmit } = useForm();
  const onSubmit = handleSubmit(
    (data) => console.log('등록하기', data),
    (data) => {
      console.log('입력 오류', data);
    }
  );
  return (
    <form onSubmit={onSubmit}>
      <FormTextFieldHookForm
        label={'배송지 명'}
        {...register('ADDRESS_NAME', {
          required: '배송지 명을 입력하세요',
          maxLength: 10,
        })}
      />
      .................
    </form>
  );
};
```

아래 코드는 입력 필드 예시인데, 실험 3에서 보았던 코드처럼 ref를 전달한다. 라이브러리 내부적으로 ref를 사용한다는 것이겠지? React 개발자 도구로 렌더링 하이라이트를 확인해보면 입력할 때마다 하이라이트 되는 부분이 없다. 공식 홈에서는 불필요한 렌더링도 없애고 성능 적으로 좋다고 자신있게 말하고 있다.

```jsx
/* 입력 필드 컴포넌트 */
const FormTextFieldHookForm = forwardRef(({ label, ...rest }, ref) => {
  return (
    <label>
      {label}
      <input ref={ref} type="text" {...rest} />
    </label>
  );
});
```

유효성 체크는 pattern으로 정의할 수 있다.

```jsx
<FormTextFieldHookForm
  label={'휴대폰 번호'}
  {...register('PHONENUMBER', {
    required: '휴대폰 번호를 입력하세요',
    maxLength: 13,
    pattern: {
      value: /010-([0-9]{4})-([0-9]{4})/g,
      message: '휴대폰 번호를 정확하게 입력해주세요',
    },
  })}
/>
```

만약 이상한 값이 들어왔다면 handleSubmit의 에러 처리 함수로 등록된 규칙과 메시지를 그대로 알려준다.

```javascript
{
  PHONENUMBER: {
    message: "휴대폰 번호를 정확하게 입력해주세요",
    type: "pattern",
    ref: input.border-2
  }
}
```
