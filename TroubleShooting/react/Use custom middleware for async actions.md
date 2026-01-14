# Use custom middleware for async actions

![[assets/images/95a5efb9ad54cabcf5b32e6cb2c4e74a_MD5.png]]

이 에러는 redux-thunk 미들웨어를 사용하지 않고 action 생성함수에서 비동기 요청을 했을 때 발생하는 에러이다.
```js
// index.js
import ReduxThunk from 'redux-thunk';

const store = createStore(
  rootReducer,
  // logger를 사용하는 경우, logger가 가장 마지막에 와야 한다.
  composeWithDevTools(applyMiddleware(ReduxThunk, logger))
); // 여러개의 미들웨어를 적용할 수 있다.
```
원래 이렇게 redux-thunk를 사용하고 있었는데 import 구문과 ReduxThunk을 빼고, 리듀서 함수에서
```js
export const increaseAsync = () => (dispatch) => {
  setTimeout(() => dispatch(increase()), 1000);
};
export const decreaseAsync = () => (dispatch) => {
  setTimeout(() => dispatch(decrease()), 1000);
};
```
요렇게 setTimeout으로 비동기 처리를 하고 발생시켰을 때 나는 에러다. 따라서 redux-thunk 라이브러리를 다운 받아 사용하도록 하자.
```sh
$ yarn add redux-thunk
``````jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import { applyMiddleware, createStore } from 'redux';
import rootReducer from './modules';
import { Provider } from 'react-redux';
import logger from 'redux-logger';
import { composeWithDevTools } from 'redux-devtools-extension';
import ReduxThunk from 'redux-thunk';

const store = createStore(
  rootReducer,
  // logger를 사용하는 경우, logger가 가장 마지막에 와야 한다.
  composeWithDevTools(applyMiddleware(ReduxThunk, logger))
); // 여러개의 미들웨어를 적용할 수 있다.

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
```
요렇게만 처리해주면, 이제 미들웨어에서 redux-thunk를 사용해 알아서 비동기 처리가 되는 모습을 볼 수 있다.
