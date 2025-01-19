# dangerouslySetInnerHTML

> 출처: https://ko.reactjs.org/docs/dom-elements.html

리액트에서는 `<div>{html}</div>` 과 같이 HTML을 그대로 렌더링 하는 형태로 JSX를 작성하면 HTML 태그가 적용되지 않고 일반 텍스트 형태로 나타나버린다. 따라서 HTML을 적용하고 싶다면 `dangerouslySetInnerHTML` 이라는 props를 설정해주어야 함.

즉, 브라우저 DOM에서 `innerTHML` 을 사용하기 위한 React의 대체 방법임.

일반적으로 코드에서 HTML을 설정하는 것은 [사이트 간 스크립팅](https://ko.wikipedia.org/wiki/사이트_간_스크립팅) 공격에 쉽게 노출될 수 있기 때문에 위험함. 따라서 React에서 직접 HTML을 설정할 수는 있지만, 위험하다는 것을 상기시키기 위해 `dangerouslySetInnerHTML`을 작성하고 `__html` 키로 객체를 전달해야 함.

```jsx
function createMarkup() {
  return { __html: 'First &middot; Second' };
}

function MyComponent() {
  return <div dangerouslySetInnerHTML={createMarkup()} />;
}
```
