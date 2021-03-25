# Portals

Potal은 부모 컴포넌트의 DOM 계층 구조 바깥에 있는 DOM 노드로 자식을 렌더링하는 최고의 방법을 제공함.

```jsx
ReactDOM.createPortal(child, container);
```

첫번째 인자(`child`)는 엘리먼트, 문자열, 혹은 fragment 와 같은 어떤 종류이든 렌더링할 수 있는 React 자식이다. 두번째 인자(`container`)는 DOM 엘리먼트다.

<br/>

## 사용법

보통 컴포넌트 렌더링 메서드에서 엘리먼트를 반환할 때 그 엘리먼트는 부모 노드에서 가장 가까운 자식으로 DOM에 마운트 된다.

```jsx
render() {
  // React는 새로운 div를 마운트하고 그 안에 자식을 렌더링 함.
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

그런데 가끔 DOM의 다른 위치에 자식을 삽입하는 것이 유용할 수 있다.

```jsx
render() {
  // React는 새로운 div을 생성하지 않고, `domNode` 안에 자식을 렌더링 함.
  // `domNode` 는 DOM 노드라면 어떠한 것이든 유효하고, DOM 내부의 어디에 있든지 상관없음.
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```

portal의 전형적인 유스케이스는 부모 컴포넌트에 `overflow: hidden` 이나 `z-index` 가 있는 경우이지만, 시각적으로 자식을 '튀어나오도록' 보여야 하는 경우도 있다. 예를 들어 다이얼로그, 호버카드나 툴팁과 같은 것.

---

📌 주의

portal을 이용하여 작업할 때 [키보드 포커스 관리](https://ko.reactjs.org/docs/accessibility.html#programmatically-managing-focus)가 매우 중요하다.

---

<br/>

## Portal을 통한 이벤트 버블링

portal이 DOM 트리의 어디에도 존재할 수 있다 하더라도 모든 다른 면에서 일반적인 React 자식처럼 동작한다. context와 같은 기능은 자식이 portal이든지 아니든지 상관 없이 정확하게 같게 동작한다. 이는 DOM 트리에서의 위치에 상관 없이 portal은 여전히 React 트리에 존재하기 때문이다.

이것에는 이벤트 버블링도 포함되어 있다. portal 내부에서 발생한 이벤트는 React 트리에 포함된 상위로 전파될 것이다. DOM 트리에서는 그 상위가 아니라 하더라도 말이다.

```html
<html>
  <body>
    <div id="app-roop"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

`#app-root` 안에 있는 `Parent` 컴포넌트는 형제 노드인 `#modal-root` 안의 컴포넌트에서 전파된 이벤트가 포착되지 않았을 경우 그것을 포착할 수 있다.

<br/>

## 예제

> 출처 : https://m.blog.naver.com/psj9102/222141597022

public 폴더에 있는 index.html 파일 수정하자.

```html
// index.html
<html>
  <head>...</head>
  <body>
    <div id="root"/>
    <div id="global-portal"/> <!-- 추가 -->
  </body>
</html> 
```

이곳이 이제 portal의 도착점이다.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Portal = ({ children }) => {
  const globalPortal = document.getElementById('global-portal');
  return ReactDOM.createPortal(children, globalPortal);
};

const App = () => {
  return (
    <div>
      <Portal>
        <div>#global portal로 이동한다</div>
      </Portal>
    </div>
  );
};

export default App;
```

![portal](https://user-images.githubusercontent.com/59427983/112404647-10968000-8d54-11eb-917e-440072fc7571.png)

이렇게 됨. useState를 사용해서 정말 잘 되는가 알아보자.

```jsx
import React, { useState } from 'react';
import ReactDOM from 'react-dom';

const Portal = ({ children }) => {
  const globalPortal = document.getElementById('global-portal');
  return ReactDOM.createPortal(children, globalPortal);
};

const App = () => {
  const [number, setNumber] = useState(0);
  return (
    <div>
      <div>{number}</div>
      <Portal>
        <button onClick={() => setNumber((pre) => pre + 1)}>증가</button>
      </Portal>
    </div>
  );
};

export default App;
```

이런식으로 잘 동작한다.