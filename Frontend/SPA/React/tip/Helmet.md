# React-helmet-async

react-helmet-async는 meta 태그를 설정해준다.

브라우저 상단에 React App 이라는 title이 표시되어 있다. 구글, 네이버 같은 검색 엔진에서 웹 페이지를 수집할 때는 `meta` 태그를 읽는다. 이 meta 태그를 리액트 앱에서 설정하는 방법을 알아보자.

```sh
$ yarn add react-helmet-async
```

<br/>

src/index.js 파일을 열어 HelmetProvider 컴포넌트로 App 컴포넌트를 감싸자.

```jsx
// src/index.js
(...)
import { HelmetProvider } from 'react-helmet-async';

(...)

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <BrowserRouter>
        <HelmetProvider>
          <App />
        </HelmetProvider>
      </BrowserRouter>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root'),
);
```

그리고 나서, `meta` 태그를 설정하고 싶은 곳에 Helmet 컴포넌트를 사용하면 된다. App을 수정해보자.

```jsx
// App.js
(...)
import { Helmet } from 'react-helmet-async';

function App() {
  return (
    <>
      <Helmet>
        <title>REACTERS</title>
      </Helmet>
      <Route component={PostListPage} path={['/@:username', '/']} exact />
      <Route component={LoginPage} path="/login" />
      <Route component={RegisterPage} path="/register" />
      <Route component={WritePage} path="/write" />
      <Route component={PostPage} path="/@:username/:postId" />
    </>
  );
}

export default App;
```

이제 title이 바뀌었다.

react-helmet-async 에서는 더 깊숙한 곳에 위치한 Helmet이 우선권을 차지한다. 예를 들어 App과 WritePage에서 Helmet을 사용할 경우, WritePage는 App 내부에 들어 있기 때문에 WritePage에서 설정하는 title 값이 나타난다.
