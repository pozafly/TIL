# vue/cli Webpack 버전 확인

vue 프로젝트에서 웹팩 버전을 확인하고 싶었다. 하지만 vue/cli 공식홈페이지에도 webpack에 대한 버전이 명시되어 있지 않았다.

<br/>

## 웹팩 옵션 속성 확인

먼저 vue/cli 로 만들어진 vue 프로젝트에서는 웹팩 옵션 속성을 볼 수 있도록 명령어가 내장되어 있다.

```shell
$ vue inspect > options.js
```

터미널 창에서 위 명령어를 치면 `option.js` 파일이 생성되고, 설정된 옵션을 해당 파일을 열어보면 볼 수 있다.

[생성된 뷰 프로젝트에 웹팩 설정 파일이 안보이는데요?](https://joshua1988.github.io/vue-camp/webpack/project-setup.html#%EC%83%9D%EC%84%B1%EB%90%9C-%EB%B7%B0-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90-%EC%9B%B9%ED%8C%A9-%EC%84%A4%EC%A0%95-%ED%8C%8C%EC%9D%BC%EC%9D%B4-%EC%95%88%EB%B3%B4%EC%9D%B4%EB%8A%94%EB%8D%B0%EC%9A%94)

<br/>

## 웹팩 버전 확인

vue 프로젝트에서 node_modules > webpack > package.json 에 보면 해당 웹팩 버전을 알 수 있다.

```json
{
  "_args": [
    [
      "webpack@4.44.2",
      "/Users/hwangsuntae/Documents/dev/Project/toy/tripllo/tripllo_vue"
    ]
  ],
  (...)
}
```

이런 식으로 웹팩 버전이 명시되어 있다.
