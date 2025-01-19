# Json-server

백엔드를 만들긴 귀찮고 빠르게 json 데이터만 임의로 받고 싶다면 json-server를 설치해서 열면 된다. 프로덕션에서 사용하기 위해 만들어진게 아니기 때문에 이 서버를 사용해 실 프로젝트를 개발하면 안된다.

## 가짜 API 서버 열기

먼저 프로젝트 디렉토리에 (src 밖에) data.json 파일을 만들고 내용을 넣자.

```json
// data.json
{
  "posts": [
    {
      "id": 1,
      "title": "리덕스 미들웨어를 배워봅시다",
      "body": "리덕스 미들웨어를 직접 만들어보면 이해하기 쉽죠."
    },
    {
      "id": 2,
      "title": "redux-thunk를 사용해봅시다",
      "body": "redux-thunk를 사용해서 비동기 작업을 처리해봅시다!"
    },
    {
      "id": 3,
      "title": "redux-saga도 사용해봅시다",
      "body": "나중엔 redux-saga를 사용해서 비동기 작업을 처리하는 방법도 배워볼 거예요."
    }
  ]
}
```

이걸 기반으로 서버를 열어보자.

```bash
$ npx json-server ./data.json --port 4000
```

또는 json-server 를 글로벌로 설치해서 다음과 같이 사용 할 수도 있다.

```bash
$ yarn global add json-server
$ json-server ./data.json --port 4000
```

json-server 를 실행하면 터미널에 다음과 같이 결과물이 뜸.

```javascript
\{^_^}/ hi!

Loading ./data.json
Done

Resources
http://localhost:4000/posts

Home
http://localhost:4000
```

그러면 우리의 가짜 API 서버가 4000 포트로 열림.

- http://localhost:4000/posts
- http://localhost:4000/posts/1

이런 식으로 조회할 수 있음.
