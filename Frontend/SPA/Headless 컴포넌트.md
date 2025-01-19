# Headless 컴포넌트

> [출처](https://www.howdy-mj.me/design/headless-components)

## Headless 란?

'머리가 없는'이라는 뜻. 구글 검색시 'Headless browser'로 창이 없는 브라우저가 가장 많이 나온다. 주로 크롤링 할 때 실제로 브라우저 창을 띄우지 않고 화면을 가상으로 렌더링 하여 실제 브라우저와 동일하게 동작하는 방식을 뜻한다.

### UI 라이브러리의 한계

외부 UI 라이브러리를 사용할 경우, 유스케이스에 맞게 기능을 새로 추가하거나 변경하고 싶어도 그에 맞게 디자인이나 기능을 수정하기가 매우 어렵다. 더 나아가 해당 라이브러리에 심각한 버그가 있거나, 유지보수를 종료한다고 하면 언젠가는 바뀌어야 한다. 그러다 결국 '그냥 컴포넌트를 만들까?' 생각이 든다.

그래서 나온 개념이 Headless UI Component로 기능은 있지만 스타일이 없는 컴포넌트를 의미한다.

<br/>

## Component 기반 UI 라이브러리 vs headless UI 라이브러리

### Component 기반 UI 라이브러리

Component 기반 UI 라이브러리는 기능과 스타일이 존재하는 라이브러리를 말하며, 대표적으로 [Material UI](https://mui.com/), [Ant Design](https://ant.design/)가 있다.

| 장점 | 바로 사용할 수 있는 마크업과 스타일이 존재<br />설정이 거의 필요 없음 |
| ---- | ------------------------------------------------------------ |
| 단점 | 마크업을 자유롭게 할 수 없음<br />스타일은 대부분 라이브러리에 있는 테마 기반으로만 변경할 수 있어 한정적임<br />큰 번들 사이즈 |

### Headless UI 라이브러리

Headless는 기능은 있지만 스타일이 없는 라이브러리로, [Headless UI](https://headlessui.dev/), [Radix UI](https://www.radix-ui.com/), [Reach UI](https://github.com/reach/reach-ui) 등이 있다.

| 장점 | 마크업과 스타일을 완벽하게 제어 가능<br />모든 스타일링 패턴 지원(ex. CSS, CSS-in-JS, UI 라이브러리 등)<br />작은 번들 사이즈 |
| ---- | ------------------------------------------------------------ |
| 단점 | 추가 설정이 필요함<br />마크업, 스타일 혹은 테마 모두 지원되지 않음 |

디자인이 그렇게 중요하지 않고, 커스텀할 곳이 많지 않다면 Component 기반 라이브러리를 사용하면 된다. 하지만 만약 반응형에 따라 디자인이 달라지고, 기능 변경이나 추가가 많이 발생한다면 Headless 라이브러리가 유지보수에 더 좋다.

<br/>

## 원칙

종류가 적기 때문에 제작해야 할 수 있다. 그럼 유지보수하기 좋은 컴포넌트를 만드는 원칙은?

컴포넌트에 어떤 메서드가 있는지 결정하기보다, 그 **컴포넌트가 무엇을 수행할 수 있는지**부터 결정해야 함. 그리고 사용자가 사용할 수 있는 기능들과 방법을 제공해야 한다. 이후에 그 기능을 어떻게 수행할 지 구현하면 된다.

중요한 점은, 기능은 어떻게 구현할지는 컴포넌트 내부에 정의하는 것으로, 외부의 다른 컴포넌트들이나 개발자가 전혀 알지 않아도 된다. 밑의 예시로 한 번 알아보자.

### Checkbox 컴포넌트 Headless로 리팩토링 하기

```tsx
import { useState } from 'react'

const Checkbox = () => {
  const [isChecked, setIsChecked] = useState(false)
  return (
    <label>
      <input
        type="checkbox"
        checked={isChecked}
        onChange={() => setIsChecked(!isChecked)}
      />
      <span>체크박스 만들기</span>
    </label>
  )
}

export default Checkbox
```

이를 다른 곳에서 바로 사용할 수 있게 하려면 어떤 내용을 체크하는 지에 대한 라벨, 체크가 되었는지의 상태 값, 체크하는 로직을 props를 받아야 한다. 수정한다면 아래와 같은 형태가 될 것이다.

```tsx
type CheckboxProps = {
  label: string
  isChecked: boolean
  onChange: () => void
}

const Checkbox = ({ label, isChecked, onChange }: CheckboxProps) => {
  return (
    <label>
      <input type="checkbox" checked={isChecked} onChange={onChange} />
      <span>{label}</span>
    </label>
  )
}

export default Checkbox
```

```tsx
export default function App() {
  const [isChecked, setIsChecked] = useState(false)
  return (
    <Checkbox
      label="체크박스 만들기"
      isChecked={isChecked}
      onChange={() => setIsChecked(!isChecked)}
    />
  )
}
```

만약 checkbox를 사용하는 곳에서 디자인과 기능이 동일하다면 이대로 사용해도 문제 없다.

만약 특정 페이지들에서 색상을 다르게 한다던지, 모바일에서는 체크 박스를 오른 쪽으로 옮겨야 한다던지 등의 레이아웃의 변경이 필요하다면 어떻게 해야 하나? 컴포넌트를 새로 만들거나 내부에서 분기 처리하면 유지보수가 점점 더 힘들어질 것이다.

그럼, 원칙을 생각해보자.

- Checkbox 컴포넌트
  - 상태에 대한 라벨(설명)이 있다.
  - 마우스로 체크 박스나 라벨을 클릭할 수 있다.
  - 체크가 안 된 상태라면 박스가 비어있고, 체크가 된 상태면 박스가 ✓ 아이콘이 생긴다.
- 사용자가 할 수 있는 기능은?
  - 사용자는 컴포넌트를 마우스로 클릭할 수 있다.

사용자는 그저 Checkbox 컴포넌트를 클릭할 뿐, 컴포넌트 내부가 어떻게 구현되어 있는지 알 수 없고, 알 필요도 없다.

<br/>

## Headless Component 만드는 방법

### 1. Compound Component

많은 라이브러리가 Compound 컴포넌트 사용. 같이 사용되는 컴포넌트들의 상태(state) 값을 공유할 수 있게 만들어주는 패턴이다.

```tsx
import * as React from 'react'

type CheckboxContextProps = {
  id: string
  isChecked: boolean
  onChange: () => void
}

type CheckboxProps = CheckboxContextProps & React.PropsWithChildren<{}>

const CheckboxContext = React.createContext<CheckboxContextProps>({
  id: '',
  isChecked: false,
  onChange: () => {},
})

const CheckboxWrapper = ({
  id,
  isChecked,
  onChange,
  children,
}: CheckboxProps) => {
  const value = {
    id,
    isChecked,
    onChange,
  }
  return (
    <CheckboxContext.Provider value={value}>
      {children}
    </CheckboxContext.Provider>
  )
}

const useCheckboxContext = () => {
  const context = React.useContext(CheckboxContext)
  return context
}

const Checkbox = ({ ...props }) => {
  const { id, isChecked, onChange } = useCheckboxContext()
  return (
    <input
      type="checkbox"
      id={id}
      checked={isChecked}
      onChange={onChange}
      {...props}
    />
  )
}

const Label = ({ children, ...props }: React.PropsWithChildren<{}>) => {
  const { id } = useCheckboxContext()
  return (
    <label htmlFor={id} {...props}>
      {children}
    </label>
  )
}

CheckboxWrapper.Checkbox = Checkbox
CheckboxWrapper.Label = Label

export default CheckboxWrapper
```

```tsx
import CheckboxWrapper from './CheckboxWrapper'

export default function App() {
  const [isChecked, setIsChecked] = useState(false)
  return (
    <CheckboxWrapper
      id="checkbox-1"
      isChecked={isChecked}
      onChange={() => setIsChecked(!isChecked)}
    >
      <CheckboxWrapper.Checkbox />
      <CheckboxWrapper.Label>체크박스 만들기</CheckboxWrapper.Label>
    </CheckboxWrapper>
  )
}
```

컴포넌트 내부에서 state를 공유하기 위해 Context API를 사용한다.

### 2. Function as Child Component

자식에 어떤 것들이 들어올지 예상할 수 없기 때문에 `children` prop으로 받아 그대로 전달하는 것이다.

```ts
type CheckboxHeadlessProps = {
  isChecked: boolean
  onChange: () => void
}

const CheckboxHeadless = (props: {
  children: (args: CheckboxHeadlessProps) => JSX.Element
}) => {
  const [isChecked, setIsChecked] = useState(false)

  if (!props.children || typeof props.children !== 'function') return null

  return props.children({
    isChecked,
    onChange: () => setIsChecked(!isChecked),
  })
}

export default CheckboxHeadless
```

```tsx
import CheckboxHeadless from './CheckboxHeadless'

export default function App() {
  return (
    <CheckboxHeadless>
      {({ isChecked, onChange }) => {
        return (
          <label>
            <input type="checkbox" checked={isChecked} onChange={onChange} />
            <span>체크박스</span>
          </label>
        )
      }}
    </CheckboxHeadless>
  )
}
```

Compound 컴포넌트보다 작성 코드량이 적다. 사용하려는 state 값을 위에서 따로 선언할 필요가 없어, 다른 컴포넌트에 해당 state를 실수로 넣을 일이 적어진다. 그리고 관련된 코드가 한 곳에 모여 있어 읽기 편하다. 하지만, 다른 곳에서 해당 state를 공유할 경우, `CheckboxHeadless` 가 감싸야 할 코드량이 많아지는 단점이 있다.

### 3. Custom hooks

```ts
import { useState } from 'react'

export const useCheckbox = () => {
  const [isChecked, setIsChecked] = useState(false)

  return {
    isChecked,
    onChange: () => setIsChecked(!isChecked),
  }
}
```

```tsx
import { useCheckbox } from './useCheckbox'

export default function App() {
  const { isChecked, onChange } = useCheckbox()
  return (
    <label>
      <input type="checkbox" checked={isChecked} onChange={onChange} />
      <span>체크박스 만들기</span>
    </label>
  )
}
```

위 두 방식 보다 간단하고 직관적이다. 하지만, state 값을 사용 되어야 하는 Checkbox 컴포넌트가 아니라 다른 곳에 작성할 실수가 발생할 수 있다.

<br/>

## 결론

Headless 컴포넌트는 스타일이 없고 로직만 존재하는 것을 뜻한다. 마크업과 스타일 수정이 자유롭기 때문에 기능 변경이 많은 곳에서 유용하다. 하지만 장단점이 명확하니 상황에 맞게 도입해야 한다.

기능은 언제든 변경될 수 있다. 따라서 어느 컴포넌트든 유지보수 하기 좋은 컴포넌트를 만들어야 한다. 유지보수 하기 좋은 컴포넌트란, 변경에 쉽게 대응할 수 있는 컴포넌트다. Headless라는 개념도 변경에 쉽게 대응하기 위해 생겨난 것이라 생각한다.
