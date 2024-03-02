# file-loader, url-loader

> [출처](https://jeonghwan-kim.github.io/series/2019/12/10/frontend-dev-env-webpack-basic.html#43-file-loader), [웹팩 공홈](https://webpack.kr/guides/asset-modules/)

webpack 5 이전에는 아래의 로더를 사용하는게 일반적이었음.

- `raw-loader` : 파일을 문자열로 가져올 때
- `url-loader` : 파일을 data URI 형식으로 번들에 인라인으로 추가할 때
- `file-loader` : 파일을 output 디렉토리로 내보낼 때

<br/>

<br/>

## LT;DR

- url-loader
  - 아이콘 처럼 용량이 작거나 자주 사용되는 녀석에게 사용하자.
  - Base64 로 인코딩 된 문자열로 빌드됨.
- file-loader
  - 네트워크 리소스 잡아먹음.
  - 파일 형태로 빌드됨.

<br/>

<br/>

## file-loader

모든 파일을 모듈로 사용하게끔 할 수 있다. 웹팩 아웃풋에 파일을 옮겨준다. 가령 CSS에서, `url()` 함수에 이미지 파일 경로를 지정할 수 있는데, 웹팩은 file-loader를 사용함.

```css
// style.css
body {
  background-image: url(bg.png);
}
```

웹팩은 엔트리 포인트인 app.js가 로딩하는 style.css 파일을 읽을 것임. 스타일 시트는 url() 함수로 bg.png를 사용하는데 로더를 작동시킴.

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/,  // png 확장자로 마치는 파일 정규식
        loader: 'file-loader',  // 파일 로더
        // options: {
        //   name: '[path][name].[ext]',
        // }
      },
    ],
  },
}
```

이 때 file-loader는 해시 값으로 파일을 만들어서 넣어준다. 즉, build시 생기는 dist 폴더에 해시 이름으로 된 png 파일이 생성되는 것임.

<img width="264" alt="스크린샷 2021-10-09 오후 6 29 22" src="https://user-images.githubusercontent.com/59427983/136652713-f809c08b-6663-45fc-9243-3e6daac8f74c.png">

요렇게. 근데, 이걸 index.html 에 있는 css 파일에서 읽으려면 실패함. 해시 값이 들어갔기 때문. 따라서, 웹팩으로 빌드한 이미지 파일은 output인 dist 폴더 아래로 이동하도록 해주고, 경로를 바로 잡게 해주어야 함. file-loader 옵션을 통해서.

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/,
        loader: 'file-loader',
        options: {
          publicPath: './dist/',  // prefix를 아웃풋 경로로 지정
          name: '[name].[ext]?[hash]',  // 파일명 형식
        }
      },
    ],
  },
}
```

`publicPath` 는 파일을 모듈로 사용할 때 경로 앞에 추가되는 문자열임. output에 설정한 dist 폴더에 이미지 파일을 옮길 것이므로, publicPath 값을 이걸로 지정함. 파일을 사용하는 측에서는 'bg.png'를 'dist/bg.png'로 변경해 사용할 것임.

근데 이렇게 해도 되고,

```js
{
  test: /\.(png|jpe?g|gif)$/i,
    loader: 'file-loader',
    options: {
      // publicPath: '/dist/', // prefix를 아웃풋 경로로 지정
      name: '[path][name].[ext]',
    },
  },
```

이렇게 해줘도 된다. 그러면 

<img width="292" alt="스크린샷 2021-10-09 오후 6 39 31" src="https://user-images.githubusercontent.com/59427983/136653013-8a80ac05-6bc6-4126-a50e-bf54d50dbf8a.png">

이런 식으로 빌드 전 src -> assets -> .png 파일이 있는 것처럼, 빌드 하면 dist 폴더에 알아서 assets 폴더 밑 png 파일이 생성된다. css, html 등에서 알아서 잘 가져오는 모습을 볼 수 있음.

<br/>

<br/>

## url-loader

사용하는 이미지 갯수가 많다면, 리소스를 사용하는 부담 + 사이트 성능에 영향을 줄 수 있다. 만약 한 페이지에서 작은 이미지를 여러 개 사용하면 Data URI Scheme를 넣는 방법이 나을 수도 있다. 이미지를 Base64로 인코딩 해 **문자열 형태**로 소스코드에 넣는 형식임.

![image](https://user-images.githubusercontent.com/59427983/136653378-dc8a8135-2c9e-4af6-95f6-5ea6061da9b3.png)

요런 식으로 들어간다.

```js
{
  test: /\.png$/,
  use: {
    loader: 'url-loader', // url 로더를 설정한다
    options: {
      publicPath: './dist/', // file-loader와 동일
      name: '[name].[ext]?[hash]', // file-loader와 동일
      limit: 5000 // 5kb 미만 파일만 data url로 처리
    }
  }
}
```

file-loader와 옵션 설정이 비슷한데 `limit` 속성만 추가 됨. 모듈로 사용한 파일 중 크기가 5kb 미만인 파일만 url-loader를 적용하는 설정. 만약 이보다 크면 file-loader가 처리하는데 옵션 중 fallback 기본 값이 file-loader 가 된다. 반면 5kb 이상인 bg.png는 옂쩐히 파일로 존재 함.

아이콘 처럼 용량이 작거나 사용 빈도가 높은 이미지는 파일을 그대로 사용하기 보다 Data URI Scheme을 적용하기 위해 url-loader를 사용하자.

<br/>

<br/>

## 두개 동시 사용

```js
module: {
  rules: [
    // file-loader로 파일로 만들어줌.
    {
      test: /\.(png|jpe?g|gif)$/i,
      loader: 'file-loader',
      options: {
        publicPath: './dist/src/assets',
        name: '[name].[ext]?[hash]',
      },
    },
    // 용량 limit 미만인 녀석을 base64로 인코딩해서 인라인으로 넣어줌.
    {
      test: /\.(png|jpe?g|gif)$/i,
      loader: 'url-loader',
      options: {
        name: '[path][name].[ext]',
        limit: 50000,
      },
    },
  ],
},
```

이렇게 file-loader와 url-loader를 둥시에 사용할 경우, build 시 하나의 이미지 파일에 두개의 빌드된 이미지가 나타나서 브라우저가 이를 제대로 처리하지 못한다. 이럴 때 사용하는게 `fallback` 이다. fallback은 url-loader에 지정해줄 수 있다.

```js
module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/i,
        loader: 'url-loader',
        options: {
          name: '[path][name].[ext]',
          limit: 50000,
          fallback: require.resolve('file-loader'),  // 요기
        },
      },
    ],
  },
```

이렇게 해주었을 경우 `limit` 가 넘는 녀석들은 file-loader로 넘겨줘서 그녀석이 처리하게 함. 즉, 모두 명시를 해주는게 아니라 url-loader 만 명시해주고 fallback을 적어주면 되겠다.