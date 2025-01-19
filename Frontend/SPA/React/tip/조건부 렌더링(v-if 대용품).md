# 조건부 렌더링(v-if 대용품)

vue에서 `v-if="로직"` 이렇게 컴포넌트를 보이고 안보이고를 할 수 있는데, react에서는 연산자를 사용해 표현한다.

<br/>

## 연산자

 && 를 활용해서 나타낼 수 있다.

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

<br/>

## 삼항연산자

```jsx
render() {
  const name = '리액트';
  return (
    <div>
      {name === '리액트' ? ( <h1>리액트 입니다.</h1> ) : ( <h2>리액트가 아닙니다.</h2> )}
    </div>
  );
}
```

이런식으로. 단, JSX에서는 if 문이 사용 금지되어 있다.

또한, setState에서 3항연산자를 사용할 때는, map을 이용한다.

```jsx
const onToggle = useCallback(
  (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, checked: !todo.checked } : todo,
      ),
    );
  },
  [todos],
);
```

이런식으로. `todo.id === id? …: …` 이라는 삼항 연산자가 사용되었다. todo.id와 현재 파라미터로 사용된 id 값이 같을 때는 정해준 규칙대로 새로운 객체를 생성하지만, id 값이 다를 때는 변화를 주지 않고 처음 받아왔던 상태 그대로 반환한다. 따라서, map을 사용해 만든 배열에서 변화가 필요한 원소만 업데이트 되고 나머지는 그대로 남아 있게 되는 것이다.
