# v-if 대용품

vue에서 `v-if="로직"` 이렇게 컴포넌트를 보이고 안보이고를 할 수 있는데, react에서는 && 를 활용해서 나타낼 수 있다.

```jsx
{onLogout && (
  <button onClick={onLogout}>
    Logout
  </button>
)}
```

이렇게. 즉, onLogout 이 있다면 button을 보여라. 그리고 이것은 javascript 문법이므로, `{` `}` 이렇게 중괄호 안에 들어 있어야 한다.

그리고 일단 개발하면서 안에 들어있는 내용을 무조건 봐야할 때가 있지.

그럴 때는 

```jsx
{true && (
  <button onClick={onLogout}>
    Logout
  </button>
)}
```

이렇게 `true` 값을 먼저 주고 개발 한 뒤 다시 조건을 주자.