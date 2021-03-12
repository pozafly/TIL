# useEffect 무한루프

소스 부터 보자.

```jsx
// App.js
import React from 'react';
import { useAxios } from './useAxios';

const App = () => {
  const { loading, data, error } = useAxios({
    url: `https://yts.mx/api/v2/list_movies.json`,
  });
  console.log(`Loading: ${loading}\nError: ${error}\nData: ${data}`);
  return (
    <div>
      <h1>Hi</h1>
    </div>
  );
};

export default App;
```

App에서 useAxios를 사용하기 위해 useAxios 커스텀 훅을 만들었다.

```jsx
// useAxios.js
import defaultAxios from 'axios';
import { useEffect, useState } from 'react';

export const useAxios = (opts, axiosInstance = defaultAxios) => {
  const [state, setState] = useState({
    loading: true,
    error: null,
    data: null,
  });
  // if (!opts.url) return;
  useEffect(() => {
    axiosInstance(opts).then((data) => {
      setState({
        ...state,
        loading: false,
        data,
      });
    });
  }, [axiosInstance, state, opts]);
  return state;
};
```

여기 useEffect 를 사용하는데, deps에 보면 `[axiosInstance, state, opts]` 이렇게 3개 의존성을 줬다. useEffect 안에서 사용하는 변수들을 react에게 거짓말 하지 않고 warning 뜨는 것으로 다 넣어준 것. (useEffect 설명에 보면 쉽게 `[]` 이것만 넣어주어 첫 랜더링 시만 불러올 수 있지만, 맹목적으로 빈 배열만 주는 것은 보통 잘못되었다고 한다.)

하지만 무한루프를 돌기 시작했다. deps안의 값을 몇개 수정해보면서, `state`, `opts` 값이 무한 루프를 돌게 만드는 것을 알게 되었다. 여기서 접근해야 할 것은

1. setState의 함수형 업데이트를 사용하여 deps를 줄이는 것
2. opts 값을 고정 값으로 주는 것

해보자.

___

## 1. setState의 함수형 업데이트를 사용하여 deps를 줄이는 것

useEffect는 `...state` 를 사용하고 있다. 선언된 상태값에 직접적으로 참고 하고 있기 때문에 deps에 명시를 해주어야 함. 우리는 이걸 없애고 싶음. 따라서 함수형 업데이트를 사용하여, react에게 상태값을 참조하지 말고 어떻게 업데이트 해야하는지를 알려주기만 하면 된다. 즉,

```js
setState((state) => ({
  ...state,
  loading: false,
  data,
}));
```

이렇게 바꾸어 주었다.

## 2. opts 값을 고정 값으로

App.js 에서

```js
useAxios({
  url: `https://yts.mx/api/v2/list_movies.json`,
});
```

이렇게 object 형태로 opts 값을 넘기고 있다. opts는 useAxios.js에서 매개변수로 받아 useEffect 안에서 적용시키고 있는 상태다. 처음에 내가 접근했던 방식은 App.js에서 url 자체를 useState를 사용하여 state 고정값으로 만든 후 넘겨주는 것. 뭐 무리없이 잘 적용되었지만, setUrl 함수를 사용할 일이 없으므로 리소스 낭비였다.

```js
// App.js
const [url, setUrl] = useState(`https://yts.mx/api/v2/list_movies.json`);
const { loading, data, error } = useAxios(url);
```

이런식으로 말이다. 그래서 그냥 const 로 주기로 함.

```jsx
// App.js
import React from 'react';
import { useAxios } from './useAxios';

const App = () => {
  const url = `https://yts.mx/api/v2/list_movies.json`;   // 고정 값
  const { loading, data, error } = useAxios(url);
  console.log(`Loading: ${loading}\nError: ${error}\nData: ${data}`);
  return (
    <div>
      <h1>Hi</h1>
    </div>
  );
};

export default App;
```

```jsx
import defaultAxios from 'axios';
import { useEffect, useState } from 'react';

export const useAxios = (opts, axiosInstance = defaultAxios) => {
  const [state, setState] = useState({
    loading: true,
    error: null,
    data: null,
  });
  // if (!opts.url) return;
  useEffect(() => {
    axiosInstance(opts).then((data) => {
      setState((state) => ({
        ...state,
        loading: false,
        data,
      }));
    });
  }, [axiosInstance, opts]);
  return state;
};
```

해결 됨. 근데 한가지 문제점은, 왜 처음과 같이 url을 객체 형태로 넘기면 가변 값이 되는 걸까? 이건 해결 안됨.