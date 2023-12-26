# modal

## ReactDOM.createPortal()

> **“Portals provide a first-class way to render children into a DOM node that exists outside the DOM hierarchy of the parent component.”**

포탈은 부모 컴포넌트의 DOM 계층 구조 바깥에 있는 DOM 노드로 자식을 렌더링하는 최고의 방법을 제공한다.

React는 트리 구조로 이루어져 있는데, 원하지 않은 경우에도 스타일이 상속되어 스타일이 꼬일 가능성이 생긴다. 하지만, Portal을 사용하면 트리구조에서 벗어나 독립적으로 관리가 가능함.

<br/>

## 사용

리액트의 html 파일이 생성된다. body 태그 내부에 `root` id를 가진 div 태그 내부에 요소가 렌더링 될 것이다. 이렇게 되면, root 아래로는 react 요소가 렌더링 될 것이고 형제 요소로 portal이 들어가게 될 것이기 때문에, 스타일 문제가 생기지 않는다.

모달을 열어줄 `createProtal` 이다.

```javascript
ReactDOM.createPortal(children, container);
```

- `children` : 렌더링할 자식 컴포넌트
- `container` : DOM element

root가 아닌 곳에 포탈을 열어주어야 하는데, `createProtal` 을 통해 넣어주자.

```html
<!doctype html>
<html lang="en">
  <head>
    <!-- ... -->
  </head>
  <body>
    <!-- 👉🏻 root-modal 추가 -->
    <div id="root-modal"></div>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

```javascript
function ModalPortal(props) {
    return ReactDOM.createPortal(props.children, document.getElementById("rood-modal");
}
```

이제 컴파운드 컴포넌트 패턴으로 Modal을 만든다. 기존에는 props로 여러 조건을 넣어주었다.

```jsx
const Modal = ({ title, content, onClose }) => {
  return (
  	<Layout>
    	{title && <Title>{title}</Title>}
      {content && <Content>{content}</Content>}
      <Footer>
        <Button onClick={onClose}>닫기</Button>
      </Footer>
    </Layout>
  );
}
```

이렇게 되면 props가 비대해지고 컴포넌트 내부가 무거워진다. 비대해질 수 있는 props를 정리하기 위해 컴포넌트를 따로 만들거나 케이스를 타입으로 받아 타입에 따른 렌더링을 해주는 시도가 있을 수 있다. 아니면 render props를 받는 방법도 있다. 예를 들면 아래와 같음.

```jsx
const Page = () => {
  const [open, setOpen] = useState(false);
  
  return (
  	<Layout>
    	<Modal open={open} render={() => (<모달내부 />)} />
    </Layout>
  );
}
```

이 방법은 어느 정도 만들어진 큰 틀에 내부를 자유롭게 커스텀할 수 있기 때문에 편할 수 있음. 다만 내부에 다시 컴포넌트를 선언해 불러와주거나 매번 새롭게 작성하는 불편함이 있겠다.

<br/>

## Compound Component Pattern

가장 큰 장점은 API의 복잡성 감소라고 할 수 있다.

패턴을 적용하면 하나의 부모 컴포넌트에 props를 이용, 하위로 내려보내는 대신, 각각 props는 가장 적합한 SubComponent에 연결되도록 구성된다.

state와 handler를 context로 관리하면서 props를 줄이면서 각각의 관심사 분리시킬 수 있다는 점이 좋다.

<br/>

## 구조

타이틀, 내용, 하단부, 레이아웃/외부영역, 모달 버튼

context에는 `React Portal`, `Context Provider` 가 들어간다.

```html
<컨텍스트>
  <버튼 />
  <포탈>
    <레이아웃>
      ...
    </레이아웃>
  </포탈>
</컨텍스트>
```

### Context

```ts
export type ModalContextType = {
  open: boolean;
  onOpen: () => void;
  onClose: () => void;
};

export const ModalContext = createContext<ModalContextType | null>(null);
```

```tsx
export default function ModalRoot({ children }: PropsWithChildren) {
  const [open, setOpen] = useState<boolean>(false);

  const onOpen = useCallback(() => {
    setOpen(true);
  }, []);

  const onClose = useCallback(() => {
    setOpen(false);
  }, []);

  return (
    <ModalContext.Provider value={{ open, onOpen, onClose }}>
      {children}
    </ModalContext.Provider>
  );
}

```

하지만 이렇게 되면 다른 곳에서도 간단하게 여닫는 로직이 많으므로 `useToggle` 훅으로 분리시키는게 좋겠다. Toast 같은 컴포넌트나 다른 컴포넌트에서도 유용하게 쓰일 것이다. (하지만 미리 만들 필요는 없음)

```ts
export const useToggle = (): ModalContextType => {
  const [open, setOpen] = useState<boolean>(false);

  const onOpen = useCallback(() => {
    setOpen(true);
  }, []);

  const onClose = useCallback(() => {
    setOpen(false);
  }, []);

  return { open, onOpen, onClose };
};

```

```tsx
import { PropsWithChildren } from 'react';
import { ModalContext } from './context/modalContext.tsx';
import { useToggle } from '../hooks/useToggle.ts';

export default function ModalRoot({ children }: PropsWithChildren) {
  const toggle = useToggle();

  return (
    <ModalContext.Provider value={toggle}>{children}</ModalContext.Provider>
  );
}

```

`ModalRoot` 상위 컴포넌트에서는 Modal을 구성하는 컴포넌트를 아직 사용하지 않는다면 이를 열어줄 방법이 없다. 버튼을 눌러 모달을 여는 경우가 아니라 서버와의 네트워크 통신 이후 자동으로 열리는 등의 케이스는 사용하기 힘들다.

관련해서 다른 상태 관리 라이브러리나 `useImperativeHandle` 을 사용하면



































































