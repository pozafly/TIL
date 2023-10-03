# html image 불러오기

웹팩 dev server를 사용하면서, html 에서 `<img src="...">`  파일을 사용해 이미지를 불러오고 싶었다. 하지만 이미지가 불러와지지 않음. 404.

현재 내 webpack.config.js 파일을 보면,

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'none',
  entry: './src/app.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, './dist'),
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      // {
      //   test: /\.(png|jpe?g|gif)$/i,
      //   loader: 'file-loader',
      //   options: {
      //     name: '[path][name].[ext]',
      //   },
      // },
      {
        test: /\.(png|jpe?g|gif)$/i,
        loader: 'url-loader',
        options: {
          name: '[path][name].[ext]',
          // publicPath: 'assets',
        },
      },
    ],
  },
  devServer: {
    ...
  },
  plugins: [
    new HtmlWebpackPlugin({
      // index.html 템플릿을 기반으로 빌드 결과물을 추가해줌
      template: 'src/index.html',
    }),
  ],
};
```

이런식으로 되어 있음. url-loader, file-loader의 차이점은 여기에 작성해뒀으니 읽어보기 바람.

어쨌든, html 파일 자체에서 img 태그로 이미지를 불러오기 위해서는 다른 문법이 필요하다.

```html
<img src=<%=require("./assets/layer-7.png").default%> alt="닝겐" />
```

이런 식으로 경로 앞에 `<%=require("경로").default&>` 라고 붙여주어야 한다. 여기서 `.default` 도 빠뜨리지 않고 넣어줘야 함. [여기](https://stackoverflow.com/questions/32753650/webpack-loading-images-from-html-templates/51924412)에서 도움을 받았다. 단, html에서 불러올 경우, url-loader 또는 file-loader 둘 중 하나는 반드시 필요하다.

원래 file-loader나, url-loader는 javascript에서 import 구문을 사용해 아래와 같이 불러올 때 이를 변환시켜주는 역할을 한다.

```js
// js
import image from './assets/image/...png';
```

이 구문을 만날 때, loader가 이를 변환시켜서 뿌려주는 역할을 하는 것이다. 하지만, require 구문 없이 html 에서 순수 경로만 적어주게 되면 해석을 하지못하고 이미지가 변환 또는 빌드 되지 않아 가져오지 못하는 것이다.

어쨌든, html에서 img 태그를 통해 이미지를 가져올 경우, loader를 설정하고, require를 사용해 가져오도록 하자.