# Window is not defined

## Window is not defined 에러

만약 위 에러를 만난다면, jsdom 관련 설정이 되지 않았기 때문. jest 환경 중, node.js 또는 브라우저 환경을 test runner로 사용하는데, vue-test-utils는 브라우저 환경인 jsdom을 사용하는 듯?

https://github.com/vuejs/vue-test-utils/issues/999#issuecomment-850273753 이 부분을 참고 했다.

<img width="863" alt="스크린샷 2021-07-08 오후 7 52 36" src="https://user-images.githubusercontent.com/59427983/124912410-e168fd80-e028-11eb-8d34-0c42eecd2239.png">

```shell
$ npm i -D jsdom
```

후에, jest.config.js 에,

```
"testEnvironment": "jsdom"
```

이렇게 넣어주고 npm t 해보자. 됨.
