# ErrorBoundary

https://react-ko.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

ErrorBoundary는 React 16부터 적용된 클래스 컴포넌트다.

class 컴포넌트로밖에 지원을 하지 않는다.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logErrorToMyService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return this.props.fallback;
    }

    return this.props.children;
  }
}
```

이렇게 정의할 수 있고,

```jsx
<ErrorBoundary fallback={<p>Something went wrong</p>}>
  <Profile />
</ErrorBoundary>
```

이렇게 적용해 사용할 수 있다.

공식문서에서 [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary)를 대신 사용할 것을 권하고 있다.

그리고 event handler에서 발생한 에러는 잡아주지 않는다. 잡지 않는 코드는 아래와 같다.

- event handler
- 비동기 코드
- 자식 컴포넌트가 아닌 에러 경계에서 발생하는 에러

<br/>

## getDerivedStateFromError, componentDidCatch

getDerivedStateFromError, componentDidCatch 두 라이프사이클 메서드를 내부적으로 사용하고 있는데 그 용도는 다르다.

- getDerivedStateFromError : error의 상태 값을 변경하는 용도
- componentDidCatch : 에러를 추적하고, sentry 등의 에러를 포착해 에러 리포트 서비스로 전송하는 용도

하지만 왜 이렇게 두개로 나눠놨을까?

[stackoverflow](https://stackoverflow.com/questions/52962851/whats-the-difference-between-getderivedstatefromerror-and-componentdidcatch)를 참고했다.

getDerivedStateFromError, componentDidCatch 둘 다, 렌더링 도중 자식 컴포넌트에서 Error가 발생하면 호출된다.

- `getDerivedStateFromError`

  - Render phase에서 호출한다.
  - render 함수를 호출하기 전에 호출한다. 따라서 error와 관련된 state를 업데이트 해주어야 한다. 자식이 먼저 렌더링 되면 안되며, fallback UI가 보여야 하기 때문이다.
  - 추가로 다른 상태 값을 업데이트 할 수도 있다. 위와 동일 이유.
  - 부수효과가 포함되면 안된다.

- `componentDidCatch`

  - Commit phase에서 호출한다.
  - error와 info(어떤 컴포넌트가 오류를 발생시켰는지) 알 수 있다.
  - 에러 리포팅 서비스에 에러를 기록할 수 있다.
  - 부수효과가 포함될 수 있다.

- 라이프사이클 메서드가 2개로 나누어진 이유는?

  [링크](https://github.com/reactjs/react.dev/pull/1223#discussion_r223174431)

  - componentDidCatch만 있을 경우 SSR 시점에 동작하지 않기 때문. SSR에는 Render phase는 있지만, Commit phase는 없기 때문이다.
  - Render phase에서 상태값 변경이 더 안전하다. 자식 컴포넌트가 그려질 때 props가 null일 수 있기 때문이다. componentDidMount, componentDidUpdate를 상위 컴포넌트가 사용하면 추가적인 오류도 발생할 수 있다.

