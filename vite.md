# Vite

회사에서 vue-cli로 만들어진 프로젝트를 Vite로 마이그레이션을 진행 중이다. [vue-cli](https://github.com/vuejs/vue-cli)는 2년 전부터 maintenance mode에 들어갔고, webpack을 기반으로 만들어졌기 때문에 번들링 및 빌드할 때 무척 느리다. vue-cli 내부에 묶여있는 eslint, prettier 등의 도구도 latest 버전과 호환이 되지 않았다.

예전부터 사용해보고 싶었던 번들러는 Vite다. Vite의 가장 큰 특징은 devServer를 ESM을 통해 제공하기 때문에 번들링을 진행하지 않고 코드를 제공한다. 그렇기 때문에 devServer가 cold-start될 때 현재 페이지에서 필요한 정보만 아주 빠르게 실행시킬 수 있다. Vite 가이드에서 가장 먼저 이야기 하고 있는 것이 ESM을 통한 devServer start 속도이다.

오늘 글은, 번들러 및 개발 환경을 vue-cli에서 Vite로 전환하며 궁금했던 개념을 약간 정리해보는 차원으로 적어보고자 한다.

<br/>

## Node.js 환경 변수

### Node.js

대부분의 bundler는 Node.js 환경 위에서 동작한다. Node.js에서는 `process` 라는 특별한 이름의 예약어를 사용해 환경 변수에 접근할 수 있다. `process` 예약어는 현재 실행되고 있는 Node.js 프로세스에 대한 정보를 가지고 있고 이를 제어할 수 있는 객체다.

대부분의 JavaScript 프레임워크에서는 `process.env` 변수를 통해 환경 변수에 접근한다. process.env는 런타임에 동적으로 변수를 추가 및 수정할 수 있다.

```js
console.log(process.env.TEST); // undefined
process.env.TEST = "test!";
console.log(process.env.TEST); // test!
```

위 코드를 Node.js의 `node` 명령어로 실행하면 주석과 같은 결과를 얻을 수 있다. 프레임워크에서는 `.env` 파일을 통해 process.env 변수에 개발자가 설정한 변수를 process.env에 주입시켜주는데, 주로 `dotenv` 라이브러리를 통해 주입한다.

### dotenv

모던 JavaScript 프레임워크에서 `dotenv` 라는 라이브러리를 내부적으로 래핑해서 사용하고 있다. [dotenv](https://github.com/motdotla/dotenv) 라이브러리는 node.js 환경 및 ESM 환경에서 `.env` 파일에 있는 환경 변수를 런타임 환경의 JavaScript로 들고올 수 있다.

라이브러리를 가볍게 분석 해보자.

```sh
$ npm init -y
$ npm i -D dotenv
```

명령어로 npm 환경을 생성한다. `package.json` 파일이 생성된 후, dotenv 라이브러리를 다운로드 받았다.

프로젝트 root에 `.env` 파일을 생성하고 테스트 값을 넣는다.

```
# .env file
TEST=ENV_TEST!
```

main.js 파일을 생성하고, dotenv 라이브러리를 통해 `.env` 파일의 환경 변수 값을 가져와보자.

```js
// src/main.js
require("dotenv").config();
console.log(process.env.TEST);  // ENV_TEST!
```

이제 `node` 명령어로 Node.js 환경에서 이를 실행시켜보자.

```sh
$ node src/main.js
=> ENV_TEST!
```

어떻게 이런 결과가 나올까? dotenv 라이브러리를 살짝 까보자. 시작은 `require("dotenv").config();` 줄이다. config 메서드를 실행한다.

```js
function config (options) {
  if (_dotenvKey(options).length === 0) {
    return DotenvModule.configDotenv(options)
  }
  // ...
}
```

`configDotenv()` 함수를 실행한다. [lib/main.js의 configDotenv 함수](https://github.com/motdotla/dotenv/blob/8076cbb4ae64b531308b42b5f27d35f49b74ed83/lib/main.js#L205). `.env` 파일의 경로를 읽고 파싱해 process 환경을 파싱된 값으로 채우고 반환한다.

``````js
function configDotenv (options) {
  let dotenvPath = path.resolve(process.cwd(), '.env')
  // ...
  const parsed = DotenvModule.parse(fs.readFileSync(dotenvPath, { encoding }))

  let processEnv = process.env
  if (options && options.processEnv != null) {
    processEnv = options.processEnv
  }

  DotenvModule.populate(processEnv, parsed, options)
  return { parsed }
}
``````

1. dotenvPath 변수에 `.env` 파일의 경로를 저장한다.

   - `process.cwd()` : node 명령을 호출한 디렉토리의 절대 경로다. /Users/pozafly/Documents/dev/play-ground/dotenv-exam
   - `path.resolve()` : 경로를 묶어 새로운 경로를 반환한다. /Users/pozafly/Documents/dev/play-ground/dotenv-exam/.env

2. parsed 변수에 `.env` 파일을 파싱해 담는다.

   - `fs.readFileSync()` : 인자로 들어오는 절대 경로로 파일 text 값을 읽어온다. 따라서, dotenvPath의 값을 읽어온다.
   - `DotenvModule.parse()` : 파싱한 text 값을 JavaScript object로 만들어서 return 한다.

   - `parsed: { TEST: 'ENV_TEST!' }` 값이 담겨있다.

3. `populate(processEnv, parsed, options)` 실행한다. populate는 '덧붙이다' 라는 뜻을 가지고 있다.

```js
function populate (processEnv, parsed, options = {}) {
  // Set process.env
  for (const key of Object.keys(parsed)) {
    // ...
    processEnv[key] = parsed[key]
  }
}
```

위에서 `processEnv = process.env` 로 레퍼런스 주소를 묶어주었기 때문에 key로 즉시 할당한다. 다시 우리가 작성했던 main.js 구문을 보자.

```js
require("dotenv").config();
console.log(process.env.TEST);  // ENV_TEST!
```

정리하면, dotenv 라이브러리는 Node.js 환경에서 `.env` 를 파싱해 text 값을 통해 객체를 생성하고, process.evn에 값을 추가해 Node.js 환경의 전역에 노출된다. JavaScript 프레임워크에서는 보통 Node.js를 한 번만 실행하기 때문에 `.env` 파일의 환경 변수 값이 변경되면 devServer를 껐다 켜주어야 값을 변경된 값으로 반영할 수 있는 이유이다.

Vite는 바로 반영이 되는데, devServer를 순간적으로 다시 시작시켜준다. .env 파일에서 변수를 변경하면 터미널 log에 `server restarted.`라고 알려준다. Vite는 webpack 처럼 모든 파일을 번들링 하지 않기 때문에 무척 빠르기에 가능하다.

dotenv에서 ESM 환경 또한 지원한다.

```js
// package.json
{
  "type": "module",
}
```

위와 같이 package.json 파일에 type을 module로 넣어주면 Node.js에서 ESM 코드를 사용할 수 있다.

> ※ Vite로 마이그레이션을 진행할 때도, package.json에 위와 같이 `"type": "module"` 을 넣어주어야 하는데, Vite는 기본적으로 ESM 방식으로 동작한다.

참고고, `main.js` 파일을, `main.mjs` 라는 확장자를 붙여주면 Node.js에서 알아서 ESM으로 동작하게 되기 때문에 package.json에 따로 붙여줄 필요는 없다.

vue-cli는 @vue/cli-services 패키지 내에서 사용하고 있고, CRA 같은 경우 react-scripts 라이브러리 내부 package.json에 `dotenv` 을 사용하고 있으며 Next.js도 내부적으로 dotenv 라이브러리를 이용해 `.env` 파일을 가져오고 있다.

> ※ [--env-file](https://nodejs.org/dist/latest-v20.x/docs/api/cli.html#--env-fileconfig)
>
> node.js 20 버전 부터는 `--env-file=config` 를 통해 `.env` 파일을 `dotenv` 라이브러리 없이 사용할 수 있도록 개발 중이다. 실제로 터미널에서 node 명령어로 `.evn` 파일을 지정해줄 수 있다. 아직은 `실험용`.
>
> ```sh
> $ node --env-file=.env --env-file=.development.env index.js
> ```
>
> ```
> // .env
> PORT=3000
> ```
>
> 따라서, 이제는 점차 `dotenv` 라이브러리가 필요없어지게 될 것 같다.

<br/>

## 모드 별 환경 변수 접근

하지만, `.env` 파일만 있는 것이 아니라 배포 환경에 따라 환경 변수의 값이 달라져야 환경 변수를 사용하는 의미가 있다. 즉, `.env.development`, `.env.production` 등의 환경 변수 파일이 추가로 필요하고 mode에 따라 지정된 값이 process.evn 값에 할당 되어 build 되어야 한다.

[dotenv-webpack](https://github.com/mrsteele/dotenv-webpack) 플러그인은 아주 간단하게 직접 path를 지정하도록 되어 있다.

먼저 사용법을 보자.

```js
// webpack.config.js
const Dotenv = require('dotenv-webpack');

module.exports = {
  plugins: [
    new Dotenv({ path: '.env.production' }),
  ],
};
```

webpack 설정 파일에 `new Dotenv()` 를 통해 인스턴스를 만든다. 인자로 path key를 가진 객체를 넣어주는데, 빌드 시 적용할 `.env.production` 파일의 경로를 적어준다. 그럼, dotenv-webpack 플러그인을 보자. [dotenv-webpack 소스](https://github.com/mrsteele/dotenv-webpack/blob/97a864d72302d25fad0cef618fcd3f09e94bc4e4/src/index.js#L22)

```js
import dotenv from 'dotenv-defaults'

class Dotenv {
  // 1.
  constructor (config = {}) {
    this.config = Object.assign({}, {
      path: './.env',
      prefix: 'process.env.'
    }, config)
  }
  
  // 3.
  getEnvs () {
    const { path } = this.config
    const env = dotenv.parse(this.loadFile({
      file: path,
    }))
    return {
      env
    }
  }
  
  gatherVariables () {
    const { env } = this.getEnvs() // 2.
    const vars = Object.assign({}, process.env); // 4.

    Object.keys(env).forEach(key => {
      const value = Object.prototype.hasOwnProperty.call(vars, key) ? vars[key] : env[key]
      vars[key] = value // 5.
    })

    return vars
  }
}
```

간추린 코드만 들고 왔다. 

1. 인스턴스를 생성할 때, 넣어준 path를 생성자에서 설정한다.
2. `getEnvs()` 메서드 실행.
3. config에 담긴 path를 가져와 `.env*` 파일을 가져오고, `dotenv` 라이브러리에 parse 메서드를 사용해 env 객체로 만들어 return.
4. `vars` 에 새로운 객체에다가 `process.evn` 값을 복사한다.
5.  `vars[key] = value` 를 통해 파싱한 값을 집어넣어 return.

즉, dotenv-webpack은 단순히 `.env` 값을 path로 받아 파싱한다.



cra -> https://github.com/facebook/create-react-app/blob/main/packages/react-scripts/config/env.js

<br/>

## Vite의 환경변수 접근

Vite는 환경 변수가 정의된 파일(`.env`)에서 여타 번들러와 마찬가지로 어플리케이션에 필요한 변수를 가져와 사용할 수 있다. Vite의 경우 ESM 환경에서 코드 조각을 들고오는데, `import.meta` 라는 것을 사용해 `.env` 파일에 접근한다.

 `import.meta` 로 node.js와 browser 모두 meta 정보에 접근할 수 있다. 근데 vite에서는 왜 dotevn 를 쓰는걸까? env를 단순히 읽어들이기 위해서일까?

vite로 플젝을 실행해서 source 탭에 트랜스파일링 된 파일 자체를 보면

```javascript
import.meta.env = {
    "VITE_TEST": "test!",
    "BASE_URL": "/",
    "MODE": "development",
    "DEV": true,
    "PROD": false,
    "SSR": false
};
```

이렇게 되어 있다. 즉, 브라우저 외부로 노출되어 있다. 따라서 어떤 변환 없이 브라우저 ESM으로 접근할 수 있다. 하지만, 서로 모듈이 다를 경우도 그렇게 되는가? 서로 어떻게 접근하게 되는가?

아래를 참조하자.

- import.meta가 proposal에 올라간 사유 : https://www.proposals.es/proposals/import.meta

- How to Access ES Module Metadata using import.meta : https://dmitripavlutin.com/javascript-import-meta/

- https://blog.bitsrc.io/understanding-import-meta-your-key-to-es-module-metadata-e2911b0d3c00
