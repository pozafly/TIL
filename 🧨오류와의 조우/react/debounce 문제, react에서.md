# debounce 문제, react에서



> [출처](https://stackoverflow.com/questions/47809666/lodash-debounce-not-working-in-react)

리액트에서 debounce를 통해 상태 값 및 리랜더링을 일으키는 일을 할 경우 제대로 동작하지 않는 경우가 있다.

리렌더링은, 일반적으로 상태값 변경을 할 때 일어나는데, 나 같은 경우 react-query의 mutation을 debounce를 통해 일으키면서 리랜더링이 일어났다.

그리고, debounce를 감싼 함수가 초기화 되면서 debounce 내부의 timeoutId 값이 초기화 되는 현상이다. timeoutId 값이 초기화가 되면, clearTimeout이 일어나지 않는다. 즉, 함수가 실행이 되버리면서, react-query의 mutation이 실행되어, 다시 리렌더링이 일어나고, 네트워크 통신이 끊임 없이 일어나는 현상이다.

```ts
export const debounce = <T extends unknown[]>(
  callback: (...params: T) => void,
  delay: number,
) => {
  let timeoutId: number;
  return (...params: T) => {
    clearTimeout(timeoutId);
    timeoutId = window.setTimeout(() => callback.apply(this, params), delay);
  };
};
```

debounce는 이렇게 생겼다. `timeoutId` 값이 초기화 된다.

그러면 리렌더링을 방지하면 된다.

---

## 해결법

[링크](https://stackoverflow.com/questions/47809666/lodash-debounce-not-working-in-react)에 따르면, debounce 함수 자체를 `useRef`를 통해 리렌더링 되더라도 다시 만들지 않게끔 권하고 있다. [이 블로그](https://rajeshnaroth.medium.com/using-throttle-and-debounce-in-a-react-function-component-5489fc3461b3)를 권한다. 하지만, 블로그 내부 댓글 링크를 따라갑보면, 

> Instead of using **useRef** or moving function outside from component, a better approach is using **useCallback**

useCallback을 권한다는 이야기가 있다. 

```js
const handleDebounceChange = debounce(saveData, 1000);
```

위 코드를

```js
const handleDebounceChange = useCallback(debounce(saveData, 1000), []);
```

이렇게 변경해주었다.

처음에는 saveData라는 함수 자체를 useCallback으로 씌웠었는데, saveData 함수는 debounce 안에 클로저로 형성되어있는 `timeoutId` 와는 상관 없음을 알게 되었고, debounce 전체를 감싸는 것이 맞다는 생각.