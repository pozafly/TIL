# modal

## ReactDOM.createPortal()

> **“Portals provide a first-class way to render children into a DOM node that exists outside the DOM hierarchy of the parent component.”**

포탈은 부모 컴포넌트의 DOM 계층 구조 바깥에 있는 DOM 노드로 자식을 렌더링하는 최고의 방법을 제공한다.

React는 트리 구조로 이루어져 있는데, 원하지 않은 경우에도 스타일이 상속되어 스타일이 꼬일 가능성이 생긴다. 하지만, Portal을 사용하면 트리구조에서 벗어나 독립적으로 관리가 가능함.

<br/>

## 사용

리액트의 html 파일이 생성된다. body 태그 내부에 `root` id를 가진 div 태그 내부에 요소가 렌더링 될 것이다. 이렇게 되면, root 아래로는 react 요소가 렌더링 될 것이고 형제 요소로 portal이 들어가게 될 것이기 때문에, 스타일 문제가 생기지 않는다.