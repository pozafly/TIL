# Error Boundary

> [출처](https://react-ko.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)

기본적으로 어플리케이션이 렌더링 도중 에러를 발생시키면 React는 화면에서 해당 UI를 제거한다. 이를 방지하기 위해 UI의 일부를 **에러 경계(Error Boundary)**로 감싸면 된다. 에러 경계는 에러가 발생한 부분 대신 에러 메세지와 같은 fallback UI를 표시할 수 있는 특수 컴포넌트다.

에러 경계 컴포넌트를 구현하려면 에러에 대한 응답으로 state를 업데이트하고 사용자에게 에러 메시지를 표시할 수 있는 [static getDerivedStateFromError](https://react-ko.dev/reference/react/Component#static-getderivedstatefromerror)를 제공해야 한다. 또한 선택적으로 [componentDidCatch](https://react-ko.dev/reference/react/Component#componentdidcatch)를 구현하여 분석 서비스에 에러를 기록하는 등 몇 가지 추가 로직을 추가할 수도 있다.

```js
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

그런 다음 컴포넌트 트리의 일부를 감쌀 수 있다:

```jsx
<ErrorBoundary fallback={<p>Something went wrong</p>}>
  <Profile />
</ErrorBoundary>
```

Profile 또는 자식 컴포넌트가 에러를 발생시키면 ErrorBoundary가 해당 에러를 '포착'하고 사용자가 제공한 에러 메시지와 함께 폴백 UI를 표시한 다음 상용 환경용 에러 리포트를 에러 리포트 서비스로 전송한다.

모든 컴포넌트를 별도의 에러 경계로 감쌀 필요는 없다. [에러 경계의 세분성](https://aweary.dev/fault-tolerance-react/)에 대해 생각할 때 에러 메시지를 표시하는 것이 합당한 위치를 고려해라. 예를 들어 메시징 앱의 경우 대화 목록 주위에 에러 경계를 배치하는 것이 좋다. 모든 개별 메시지 주위에 에러 경계를 배치하는 것도 좋다. 하지만 모든 아바타 주위에 경계를 설정하는 것은 적절하지 않다.

> 현재 에러 경계를 함수 컴포넌트로 작성할 수 있는 방법은 없다. 하지만 에러 경계 클래스를 직접 작성할 필요는 없다. 예를 들어, [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary)를 대신 사용할 수 있음.

