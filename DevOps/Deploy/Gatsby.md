# Gatsby

개츠비를 Github pages에 배포해보자. 기존에 있던 jekyll 기반 블로그는 repo이름을 변경해뒀다.

> 출처: [Gatsby로 Github page 만들기](https://velog.io/@_woogie/Gatsby%EB%A1%9C-Github-page-%EB%A7%8C%EB%93%A4%EA%B8%B0), [github blog 배포하기(gh-pages & surge)](https://velog.io/@hwang-eunji/github-blog-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0), [Gatsby Github Page 배포하기](https://velog.io/@jwisgenius/Gatsby-Github-Page-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0)

<br/>

## Gatsby 설치

먼저, node가 깔려 있어야 함.

```sh
$ npm i -g gatsby-cli
```

gatsby cli가 전역으로 필요함. 깔아주자. 나는 고스트 블로그 테마가 마음에 들어서 이것으로 함.

```sh
$ gatsby new gatsby_blog https://github.com/scttcper/gatsby-casper
```

그러면 폴더가 생기고, 로컬에서 돌려볼려면,

```sh
$ gatsby develop
```

하면 기본적으로 localhost:8000 에 뜸.

<br/>

## Github page 배포

Github에 [사용자이름].github.io 라는 레파지토리를 만들어서 소스를 올리면 되는데, 개츠비는 추가적인 설정이 조금 더 필요함.

그냥 배포하기만 하면 개츠비의 readme.md 파일만 보일 것이다. 왜냐면 배포된 파일에 `index.js` 파일 기준으로 github page가 이해하고 분석하기 때문에. 이게 없으면 리드미만 보여줌.

따라서 브랜치를 하나 파서, 빌드만 하면 알아서 자동으로 master 브랜치로 update가 되어 페이지를 보여주도록 해주어야 함.

```bash
$ git checkout -b source master
```

`source` 라는 브랜치를 만들었다. master 기반으로. 그리고 여기서 작업을 해줄 것임.

```bash
$ npm i -D gh-pages
```

요걸 깔아주자. 그담 `package.json` 에 가자.

```json
{
  "homepage" : "https://<깃헙아이디>.github.io/"
  "script" : {
  	// ...
    "deploy" : "gatsby build && gh-pages -d public -b master"
}
```

요렇게 해주자. 그러면 이제 source 브랜치에서

```bash
$ npm deploy
$ yarn deploy
```

둘 중 하나 해주면 알아서 master 브랜치로 빌드한 내용이 올라가게 되고 정상적으로 페이지가 뜨는걸 볼 수 있다.

<br/>

추가로, 배포 에러가 뜬다면, yarn deploy를 하면 에러나는 경우, 스타터의 react, react-dom, gatsby를 다시 설치하고 `gatsby clean` 을 해주면 됨. -> 버전 문제.

가장 중요한 사실은

1. public폴더에 index.html이 들어있고,
2. public폴더는 deploy하면 자동 생성이되고,
3. github는 root폴더에 index.html이 없으면 readme.md, 이것도 없으면 404를 보낸다.
