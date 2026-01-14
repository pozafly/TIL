# CORS 설정

브라우저에서 기본적으로 API를 요청할 때 브라우저의 현재 주소와 API의 주소의 도메인이 일치해야만 데이터를 접근할 수 있게 되어 있다. 만약 다른 도메인에서 API를 요청해서 사용할 수 있게 해주려면 [CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS#HTTP_%EC%9D%91%EB%8B%B5_%ED%97%A4%EB%8D%94) 설정이 필요하다.

json-server로 만든 서버의 경우에는 모든 도메인을 허용해주는 CORS 규칙이 적용되어 있다. 하지만 우리가 Open API를 만드는게 아니라면 모든 도메인을 허용하면 안된다. 하게 된다면 특정 도메인만 허용 해주어야 함.

실제 서비스를 개발하게 될 때는 서버의 API를 요청해야할 때, 기본적으로는 localhost:3000 에서 들어오는 것이 차단되기 때문에 서버 쪽에 해당 도메인을 허용하도록 구현해야 한다. 백엔드 개발자가 따로 있다면 백엔드에게 해당 도메인을 허용해달라 요청해야함. 근데 그럴 필요 없음. 웹팩 개발서버에서 제공하는 Proxy라는 기능이 있기 때문.

<br/>

## Proxy 설정하기

![[assets/images/9925a28e3923bb7fb96815eb62e64320_MD5.png]]

웹팩 개발서버의 프록시를 사용하게 되면, 브라우저 API를 요청할 때 백엔드 서버에 직접적으로 요청을 하지 않고, 현재 개발서버의 주소로 요청하게 됨. 그러면 웹팩 개발 서버에서 해당 요청을 받아 그대로 백엔드 서버로 전달하고, 백엔드 서버에서 응답한 내용을 다시 브라우저쪽으로 반환한다. 웹팩 개발서버의 proxy 설정은 원래 웹팩 설정을 통해 적용하지만, CRA를 통해 만든 리액트 프로젝트에서는 package.json 에서 `"proxy"` 값을 설정해 쉽게 적용할 수 있다. ([참고](https://create-react-app.dev/docs/proxying-api-requests-in-development/))

package.json을 열어 맨 아래에 `"proxy"` 값을 설정해보자.

```json
// package.json
  (...),
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "proxy": "http://localhost:4000"   // 이부분
}
```

그 다음엔 api/posts.js 파일을 열어 http://localhost:4000/posts 대신, /posts URL 로 요청하도록 하면 됨. 이렇게 요청하는 주소의 도메인이 생략된 경우에는 현재 페이지의 도메인 (지금의 경우 http://localhost:3000) 을 가르키게 된다.

```js
// api/posts.js
import axios from 'axios';

export const getPosts = async () => {
  const response = await axios.get('/posts');
  return response.data;
};

export const getPostById = async id => {
  const response = await axios.get(`/posts/${id}`);
  return response.data;
};
```

이제 개발서버를 다시 시작하고, Network 탭을 보면 localhost:4000 이 아닌, localhost:3000으로 요청했다.

나중에 배포 시, 리액트로 만든 서비스와 API가 동일한 도메인에서 제공되는 경우 이대로 계속 진행하면 되지만 만약 API의 도메인과 서비스의 도메인이 다르다면, (예: 서비스는 velog.io, API는 api.velog.io), axios의 글로벌 baseURL 을 설정하면 된다.

```js
axios.defaults.baseURL = process.env.NODE_ENV === 'development' ? '/' : 'https://api.velog.io/';
```

`process.env.NODE_ENV`는 현재 환경이 프로덕션인지 개발모드인지 볼 수 있는 값. ([참고](https://create-react-app.dev/docs/adding-custom-environment-variables))

위와 같은 설정을 만약 하게 된다면, 개발 환경에선 프록시 서버 쪽으로 요청하고, 프로덕션에선 실제 API 서버로 요청을 하게 됩니다. 이러한 설정은 아까 언급한것처럼 API 도메인이 실서비스의 도메인과 다를 때만 하면 됨.

프로젝트를 개발 할 때 proxy 를 사용하는 것은 필수적인건 아니지만, 사용하게 되면 백엔드 쪽에서 불필요한 코드를 아낄 수 있으니 (백엔드에서 개발서버를 위한 CORS 설정을 안해도 되니까) 꽤나 유용한 기능.
