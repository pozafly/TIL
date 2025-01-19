# 고차 컴포넌트(HOC)

> 출처: https://velopert.com/3537

코드를 작성하다보면, 반복해서 작성하게 되는 코드들이 있다. 우리는 주로 그런 것들을 함수화 해 재사용하곤 한다. 컴포넌트 또한 비슷하다. 같은 UI 관련 코드가 재사용될 수 있다면 우리는 컴포넌트를 만들어 컴포넌트를 재사용 함. 근데 컴포넌트 기능 상에서도, 자주 반복되는 코드들이 나타날 수 있다.

리액트 컴포넌트를 작성하게 될 때 반복될 수 있는 코드들은 HOC를 만들어 해결해줄 수 있다. HOC는 하나의 함수다. 함수를 통해 컴포넌트에 특정 기능을 부여한다.

<br/>

## 반복되는 코드 발견하기

```jsx
// Post.jsx
import axios from 'axios';
import React, { useEffect, useState } from 'react';

const Post = () => {
  const [data, setData] = useState({ data: null });

  const initialize = async () => {
    try {
      const response = await axios.get('https://jsonplaceholder.typicode.com/posts/1');
      setData({ data: response.data });
    } catch (e) {
      console.log(e);
    }
  };

  useEffect(() => {
    initialize();
  }, []);

  return (
    <div>
      <pre>{JSON.stringify(data.data)}</pre>
    </div>
  );
};

export default Post;
```

이 컴포넌트에서는, 특정 주소에 GET 요청을 날리고, 결과물을 state의 data 안에 담는다. 그리고, 해당 data를 JSON 형태 그대로 렌더링 해준다.

Comments.jsx 도 확인해보자.

```jsx
// Comments.jsx
import axios from 'axios';
import React, { useEffect, useState } from 'react';

const Common = () => {
  const [data, setData] = useState({ data: null });

  const initialize = async () => {
    try {
      const response = await axios.get(
        'https://jsonplaceholder.typicode.com/comments?postId=1'
      );
      setData({ data: response.data });
    } catch (e) {
      console.log(e);
    }
  };

  useEffect(() => {
    initialize();
  }, []);

  return <div>{JSON.stringify(data.data)}</div>;
};

export default Common;
```

axios의 주소만 빼고 다 같다. 우선, 이렇게 반복되는 코드를 발견하는 것이 첫번째 단추임.

<br/>

## HOC 작성하기

이 반복되는 코드를 없애기 위해 하나의 함수를 작성하자. 주로 HOC 이름을 만들 때, `with___` 형식으로 짓는다. 예를 들어 웹 요청을 하는 HOC를 만들테니 withRequest라고 지어주도록 하자.

HOC 원리는, 파라미터로 컴포넌트를 받아오고, 함수 내부에서 새 컴포넌트를 만든 다음에 해당 컴포넌트 안에서 파라미터로 받아온 컴포넌트를 렌더링 하는 것이다. 그리고, 자신이 받아온 props 들은 그대로 파라미터로 받아온 컴포넌트에게 다시 주입해주고, 필요에 따라 추가 props도 넣어둔다.(예를 들면 웹 요청 결과물.)

```jsx
// withRequest.jsx
import React from 'react';

const WithRequest = (url) => (WrapperedComponent) => {
  return () => {
    return <WrapperedComponent {...url} />;
  };
};

export default WithRequest;
```

위 코드를 보면, 함수에서 또 다른 함수를 리턴하도록 했다. (url, WrappedComponent) 형식이 아니라, `(url) => (WrappedComponent)` 로 한 이유는, 나중에 여러개의 HOC 를 합쳐서 사용하게 될 때 더욱 편하게 사용하기 위함.

그럼, HOC에 기능을 붙여보자.

```jsx
import axios from 'axios';
import React, { useEffect, useState } from 'react';

const withRequest = (url) => (WrapperedComponent) => {
  // 대문자로 시작 안하면, Lint 에서 에러 발생 (compile error)
  // https://kimxavi.tistory.com/entry/react-HOC-Functional-Component
  const Hoc = (props) => {  // 🌈
    const [data, setData] = useState({ data: null });
    const initialize = async () => {
      try {
        const response = await axios.get(url);  // 🔥
        setData({ data: response.data });
      } catch (e) {
        console.log(e);
      }
    };

    useEffect(() => {
      initialize();
    }, []);

    return <WrapperedComponent {...props} data={data} />;  // 💧
  };
  return Hoc;
};

export default withRequest;
```

- 🌈: Hoc 함수는 대문자로 시작해야 함. React function 컴포넌트이기 때문. 그리고, props를 받아왔음.
- 🔥: 매개변수로 url을 받아온 것을 넣어준다.
- 💧: props를 넣어줬음. 그리고 axios 를 통해 받은 data를 파라미터로 받은 컴포넌트에 넣어주도록 설정했다.

<br/>

## HOC 사용하기

Post, Comments 에 적용해보자.

```jsx
import React from 'react';
import withRequest from './withRequest';

const Post = ({ data }) => {  // 💧
  return <div>{JSON.stringify(data.data)}</div>;
};

export default withRequest('https://jsonplaceholder.typicode.com/posts/1')(Post);  // 🌈
```

- 💧: props로 data를 받아온다.
- 🌈: 컴포넌트를 내보낼 때, 이렇게 매개변수에 넣어주면 된다.

또는,

```jsx
const PostWithData = withRequest('https://jsonplaceholder.typicode.com/posts/1')(Post);
export default PostWithData
```

이렇게 내보내 줘도 됨. 마찬가지로 comments.jsx에도 이런 형식으로 만들어주면 되겠다.
