# Redux vs context API

react-redux는 내부적으로 Context API를 사용하고 있다. 그리고 children props로 하위 컴포넌트를 받고 있다.

아래는 redux의 전역 store 설치 로직이다.

```jsx
import store from './app/store'
import { Provider } from 'react-redux'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

어플리케이션 전체에 입히는 형태다. 그러면 Provider로 children이 App으로 설정된다. 아래는 [react-redux 코드](https://github.com/reduxjs/react-redux/blob/master/src/components/Provider.tsx#L61)임.

````jsx
function Provider<A extends Action = AnyAction, S = unknown>({
  store,
  context,
  // ...
}: ProviderProps<A, S>) {
  const contextValue = useMemo(() => {
    // ...
  }, [store, serverState])
  const previousState = useMemo(() => store.getState(), [store])

  useIsomorphicLayoutEffect(() => {
    // ...
  }, [contextValue, previousState])
  const Context = context || ReactReduxContext
  
  return <Context.Provider value={contextValue}>{children}</Context.Provider>
}
````

Provider의 value를 useMemo로 감싸고, `{children}`을 사용하면 하위 컴포넌트가 렌더링 되지 않는다. [링크](https://kattya.dev/articles/2021-04-17-fixing-re-renders-when-using-context-in-react/)

[원리](https://kentcdodds.com/blog/optimize-react-re-renders)는 이곳에 있다. 상위의 props가 바뀌면 상위 컴포넌트는 리랜더링이 일어나지만, {children}으로 전달 된 컴포넌트는 Provider 컴포넌트 내부에 선언 된 것이 아니라 `{children}` 으로 전달 되기만 했고, 동일 위치에 동일 props로 렌더링 되었기 때문에 상위 상태 값이 변경 되어도 리랜더링 되지 않는다.

https://yrnana.dev/post/2021-08-21-context-api-redux/ 요기에도 잘 나와 있다. https://medium.com/@bhavyasaggi/how-did-i-re-render-sharing-state-through-react-context-f271d5890a7b 이곳에도.

어쨌든, 위와 같이 react-redux에서는 저런 방식으로 하위 컴포넌트들의 리랜더링을 방지하고 있다.

[재조정 VDOM](https://www.youtube.com/watch?v=BYbgopx44vo)과 같은 곳도 잘 표현해두었음.
