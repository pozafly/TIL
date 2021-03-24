# Portals

Potal은 부모 컴포넌트의 DOM 계층 구조 바깥에 있는 DOM 노드로 자식을 렌더링하는 최고의 방법을 제공함.

```jsx
ReactDOM.createPortal(child, container);
```

첫번째 인자(`child`)는 엘리먼트, 문자열, 혹은 fragment 와 같은 어떤 종류이든 