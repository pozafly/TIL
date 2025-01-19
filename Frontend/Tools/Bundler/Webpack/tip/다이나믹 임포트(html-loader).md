# 다이나믹 임포트(html-loader)

webpack은 기본적으로 `index.html` 파일 하나만 허용함. 물론 multiple entry를 사용할 수도 있겠지만, 내가 지금 필요한 상황은

1. html 파일 자체의 text가 필요함.
2. 동적으로 html 파일 text를 Javascript에 가져와야 함.

따라서 동적 import를 사용해 html 파일안에 있는 text를 가져와보자.

<br/>

## Html-loader

webpack dev server를 사용하고 있다면 `import` 구문을 사용해 html 파일을 가져올 수 있다. 그러려면 `html-loader` 설정을 해줘야 함.

```shell
$ npm i -D html-loader
```

먼저 깔아준 후,

webpack.config.js

```js
module: {
  rules: [
    {
      test: /\.html$/i,
      loader: 'html-loader',
      options: {
        // 🌈
        esModule: false,
      },
    },
    (...)
  ]
}
```

이렇게 설정해주자. 만약 옵션에 🌈 위와 같이 `esModule` 이 설정되어 있지 않다면, png나 다른 파일이 file-loader로 부터 가져오는 경로가 바뀌게 될 것이다. 주의하시길. 가장 하단에 webpack.config.js 파일 full version을 올려두겠다.

어쨌든 이렇게 설정해주면, html 파일 자체의 text를 import 할 수 있는 조건이 완성되었다.

<br/>

## 동적 import

Javascript 파일 내에서 보통은 `import` 구문을 상단에 표시하곤 하는데, if 문 내에서나, 아니면 문자열로 가져올 파일 이름을 동적으로 매칭시켜 가져오도록 할 수 있다.

```js
import(`./book/${liurl}`).then((result) => {
  const htmlText = result.default;
  console.log(htmlText);
});
```

이런 식으로 사용한다. 즉 내가 필요한 것은 book 폴더 안에 있는 `책이름.html` 이다. 이 때, 동적 import는 Promise를 사용하고, Promise가 끝나면, then으로 받아 해당 html 파일 내용을 Javascript로 가져올 수 있게 된다.

---

참고로 제 webpack.config.js 의 전체 내용이다.

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'none',
  entry: './src/app.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.html$/i,
        loader: 'html-loader',
        options: {
          esModule: false,
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|jpe?g|gif)$/i,
        loader: 'file-loader',
        options: {
          name: '[path][name].[ext]',
        },
      },
    ],
  },
  devServer: {
    port: 8080,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        // changeOrigin: true,
      },
    },
  },
  plugins: [
    new HtmlWebpackPlugin({
      // index.html 템플릿을 기반으로 빌드 결과물을 추가해줌
      template: 'src/index.html',
    }),
  ],
};
```
