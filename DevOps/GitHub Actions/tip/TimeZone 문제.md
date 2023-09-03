# TimeZone 문제

> [출처](https://github.com/vercel/next.js/discussions/39425)

리액트를 사용하는 Gatsby에서 배포하면 렌더링이 두번 되면서 위와 같은 에러가 발생함

```jsx
react-dom.production.min.js:119 Uncaught Error: Minified React error #425; visit <https://reactjs.org/docs/error-decoder.html?invariant=425> for the full message or use the non-minified dev environment for full errors and additional helpful warnings. at d7 (react-dom.production.min.js:119:38) at gt (react-dom.production.min.js:206:80) at h6 (react-dom.production.min.js:281:80) at h5 (react-dom.production.min.js:280:438) at h3 (react-dom.production.min.js:280:315) at h2 (react-dom.production.min.js:280:172) at hT (react-dom.production.min.js:268:193) at A (scheduler.production.min.js:14:181) at MessagePort.g (scheduler.production.min.js:15:101)

react-dom.production.min.js:148 Uncaught Error: Minified React error #418; visit <https://reactjs.org/docs/error-decoder.html?invariant=418> for the full message or use the non-minified dev environment for full errors and additional helpful warnings. at fp (react-dom.production.min.js:148:277) at _ (react-dom.production.min.js:293:444) at h5 (react-dom.production.min.js:280:375) at h3 (react-dom.production.min.js:280:315) at h2 (react-dom.production.min.js:280:172) at hT (react-dom.production.min.js:268:193) at A (scheduler.production.min.js:14:181) at MessagePort.g (scheduler.production.min.js:15:101)

react-dom.production.min.js:293 Uncaught Error: Minified React error #423; visit <https://reactjs.org/docs/error-decoder.html?invariant=423> for the full message or use the non-minified dev environment for full errors and additional helpful warnings. at _ (react-dom.production.min.js:293:137) at h5 (react-dom.production.min.js:280:375) at h3 (react-dom.production.min.js:280:315) at h2 (react-dom.production.min.js:280:172) at hU (react-dom.production.min.js:271:69) at hT (react-dom.production.min.js:268:409) at A (scheduler.production.min.js:14:181) at MessagePort.g (scheduler.production.min.js:15:101)
```

이는 서버에서 렌더링한 것과 클라에서 렌더링 것  차이가 날 경우 발생함.

링크에 보면, date-fns에서 빌드시 타임 존 기준으로 빌드되어 다르게 html 파일이 만들어짐.

따라서 배포된 클라 환경에서 react가 표시할 때 빌드 환경의 날짜 기준으로 빌드함.

따라서, GitHub Actions 러너의 linux에서 time-zone을 맞춰주고 빌드를 해야 위와 같은 현상이 발생하지 않음.

```
steps:
  # time zone이 세팅 되지 않으면 배포 시 ReactDOM 관련 Error 발생.
  # GitHub Actions linux 환경에서 Gatsby 빌드 시 time zone 맞추는 작업 필요.
  - name: Time zone setting
    run: sudo timedatectl set-timezone Asia/Seoul
  - uses: actions/checkout@v3
```

step에 가장 처음 run에 적힌 명령어로 타임존을 서울로 맞춰주고 빌드하면 아무 문제가 생기지 않는다.