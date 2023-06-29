# Portals

> [공식 문서](https://ko.legacy.reactjs.org/docs/portals.html)

Portal은 부모 컴포넌트의 DOM 계층 구조 바깥에 있는 DOM 노드로 자식을 렌더링하는 방법이다.

```js
ReactDOM.createPortal(child, container);
```

- 첫번째 인자 (child)는 엘리먼트, 문자열, 혹은 frgment와 같은 어떤 종류이든 렌더링할 수 있는 React 자식이다.
- 두번째 인자(container)는 DOM 엘리먼트다.

## 사용법

보통 컴포넌트 렌더링 메서드에서 엘리먼트를 반환할 때 그 엘리먼트는 부모 노드에서 가장 가까운 자식으로 DOM에 마운트 된다.

```jsx
render() {
  // React는 새로운 div를 마운트하고 그 안에 자식을 렌더링한다.
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
  // React는 새로운 div를 생성하지 않고 `domNode` 안에 자식을 렌더링한다.
  // `domNode`는 DOM 노드라면 어떠한 것이든 유효하고, 그것은 DOM 내부의 어디에 있든지 상관 없다.
  return ReactDOM.createPortal(
  	this.props.children,
    domNode,
  );
}
```

portal의 전형적 유스케이스는 부모 컴포넌트에 `overflow: hidden` 이나 `z-index`가 있는 경우이지만, 시각적으로 자식을 '튀어나오도록' 보여야 하는 경우도 있다. 예를 들어, 다이얼로그, 호버카드나 툴팁과 같은 것이다.

> 📌 주의
>
> portal을 이용해 작업할 때 [키보드 포커스 관리](https://ko.legacy.reactjs.org/docs/accessibility.html#programmatically-managing-focus)가 매우 중요하다

## portal을 통한 이벤트 버블링

portal이 DOM 트리의 어디에도 존재할 수 있다 하더라도 모든 다른 면에서 일반적인 React 자식처럼 동작한다. context와 같은 기능은 자식이 portal 이든지 아니든지 상관 없이 정확하게 같게 동작한다. -> 즉, React 하위 자식처럼 동작한다.

DOM 트리에서의 위치에 상관없이 portal은 여전히 React 트리에 존재하기 때문이다.

이것은 이벤트 버블링도 포함되어 있다. portal 내부에서 발생한 이벤트는 React 트리에 포함된 상위로 전파될 것이다. DOM 트리에서는 그 상위가 아니라 하더라도 말이다. 다음 HTML 구조를 가정해보자.

```html
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

`#app-root` 안에 있는 `Parent` 컴포넌트는 형제 노드인 `#modal-root` 안의 컴포넌트에서 전파된 이벤트가 포착되지 않았을 경우 그것을 포착할 수 있다.

## 오해

> [출처](https://jeonghwan-kim.github.io/2022/06/02/react-portal)

어플리케이션을 이동시키는 기능이다. 어플리케이션이 마운트 되는 위치를 이동한다는 의미다. 루트 돔 트리를 벗어나 다른 돔에서 어플리케이션의 일부분을 그릴 수 있다. 팝업이나, 툴팁처럼 현재 UI 문맥과 다를 경우 포탈을 사용한다.

## 포탈에 대한 오해

어플리케이션이 동작하는 돔 외부에 일부 기능을 옮겨놓으면 성능상 낫다는 오해다. 가령, 모달은 UI 컨텍스트가 다르기 때문에 메인 어플리케이션의 UI에 영향을 주지 않기 위해 포탈을 사용한다고 생각했다. 다른 돔에서 모달 엘리먼트가 변하더라도 메인 어플리케이션이 돌아가는 돔에는 영향을 주지 않을 것이라는 생각.

하지만, 이것이 포탈이 해결하려는 주된 문제는 아니다. UI 트리 하단에 모달 엘리먼트가 있더라도 이것이 전체 돔에 영향을 주는 것은 아니다. 오리혀 트리 말단에 있기 때문에 모달을 그리는 돔만 변할 것이다.

```jsx
const PortalModal = props => {
  // 모달이 마운트 될 엘리먼트. 루트 엘리먼트와 다른 곳이다.
  const modalRoot = document.querySelector("#modal-root")
  // 모달 앨리먼트를 modalRoot에 마운트할 것이다.
  return ReactDOM.createPortal(<Modal {...props} />, modalRoot)
}
```

```html
<!-- 어플리케이션이 마운트될 위치 -->
<div id="root"></div>
<!-- 모달이 마운트될 위치 -->
<div id="modal-root"></div>
```

```jsx
const App = () => (
  <div className="App">
    App
    <PortalModal>Modal</PortalModal>
  </div>
)
```

App에서 ProtalModal을 불렀기 때문에 `#modal-root` 에 렌더링 된다고 하더라도 이벤트 버블링은 App까지 올라올 것이다.

```jsx
const Modal = ({ children }) => (
  <div className="Modal">
    {children}
    {/* 이 버튼을 클릭하면 이벤트가 부모로 버블링될 것이다. */}
    <button>Button</button>
  </div>
)

const ProtalModal = () => React.DOM.createPortal(/* ... */)

const App = () => {
  const handleClick = e => {
    // button에서 발생한 이벤트가 여기서 잡힐 것이다.
    console.log(e.target) // <button>
  }
  return (
    <div className="App" onClick={handleClick}>
      App
      <PortalModal>Modal</PortalModal>
    </div>
  )
}
```

App은 이 이벤트를 잡아서 handleClick에서 처리할 것이다. 실제 클릭해보면 이벤트가 잘 잡힌다.

앨리먼트가 다른 돔을 사용하더라도 포탈은 어플리케이션 트리 모양대로 이벤트를 전파한다.

> 이는 DOM 트리에서의 위치에 상관없이 portal은 여전히 React 트리에 존재하기 때문입니다.

## 결론

리액트 포탈은 어플리케이션의 메인 돔 외부에 앨리먼트 일부를 그리기 위한 기능이다. 엘리먼트를 다른 DOM으로 옮겨 CSS 상속 구조에서 벗어나기 위함이다.

포탈로 만든 엘리먼트는 실제 DOM 구조와 달리 리액트 어플리케이션 컴포넌트 트리 구조를 따른다. 이 순서대로 이벤트가 전파되는 것.