# npm 호환성

The engine "node" is incompatible with this module

<br/>

<img width="665" alt="스크린샷 2021-02-25 오후 6 52 28" src="https://user-images.githubusercontent.com/59427983/109136041-0b690400-779b-11eb-8630-1b00f1c630b4.png">

호환성 문제인 듯 싶다. yarn에서 create-react-app을 했을 때 만남. 아마도 postcss와 npm 버전 호환 문제인 듯.

node를 업뎃 해보자.

```shell
$ sudo npm cache clean -f # 강제캐시삭제
$ sudo npm install -g n # n 모듈 설치
$ sudo n stable # or sudo n 12.14.0 (버전명)
$ node -v # 버전 확인
```

해결!