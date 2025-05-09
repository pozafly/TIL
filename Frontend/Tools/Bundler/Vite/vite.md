# Vite에서 import.meta는 왜 사용하는걸까? (feat. HMR)

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

### Dotenv

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

   - `process.cwd()`: node 명령을 호출한 디렉토리의 절대 경로다. /Users/pozafly/Documents/dev/play-ground/dotenv-exam
   - `path.resolve()`: 경로를 묶어 새로운 경로를 반환한다. /Users/pozafly/Documents/dev/play-ground/dotenv-exam/.env

2. parsed 변수에 `.env` 파일을 파싱해 담는다.

   - `fs.readFileSync()`: 인자로 들어오는 절대 경로로 파일 text 값을 읽어온다. 따라서, dotenvPath의 값을 읽어온다.
   - `DotenvModule.parse()`: 파싱한 text 값을 JavaScript object로 만들어서 return 한다.
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

Vite는 바로 반영이 되는데, devServer를 순간적으로 다시 시작시켜준다..env 파일에서 변수를 변경하면 터미널 log에 `server restarted.`라고 알려준다. Vite는 webpack 처럼 모든 파일을 번들링 하지 않기 때문에 무척 빠르기에 가능하다.

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
5. `vars[key] = value` 를 통해 파싱한 값을 집어넣어 return.

즉, dotenv-webpack은 단순히 `.env` 값을 path로 받아 파싱한다.

<br/>

## Vite의 환경변수 접근

Vite는 환경 변수가 정의된 파일(`.env`)에서 여타 번들러와 마찬가지로 어플리케이션에 필요한 변수를 가져와 사용할 수 있다. Vite의 경우 [공식 문서에서 환경 변수 접근법](https://ko.vitejs.dev/guide/env-and-mode.html)에 대해 `import.meta` 라는 JavaScript 문법으로 접근할 것을 알려주고 있다. `.env` 파일에 환경과 관련된 변수를 `VITE_` prefix를 주고, 코드 상에서 `import.meta.env` 로 선언한 값을 들고올 수 있다.

### import.meta

[`import.meta`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/import.meta) 라는 JavaScript 문법을 공부하며 MDN 문서에 아직 아무도 번역하지 않아 이번 기회에 함께 번역해보았다. `import.meta` 는 ES Module이 탄생하면서 모듈에 대한 메타 데이터(부수 데이터)를 저장하기 위한 객체다.

여기서 말하는 메타 데이터란, 이미지 파일을 생각하면 조금 더 이해하기 쉽다. 휴대폰으로 사진을 찍으면 사진의 메타 데이터에 '날짜', '사진 용량', '디바이스 정보', '위치 데이터' 등의 메타 데이터가 담기게 된다. `import.meta` 는 '모듈' 자체에 대한 메타 데이터를 담고 있다.

import.meta는 대표적으로 모듈 메타 데이터인 경로(URL)를 담고 있는데, 이는 호스트 환경(브라우저 혹은 Node.js)에 따라 저장되는 내용이 달라진다. 먼저 Node.js를 통해 import.meta가 가지고 있는 정보를 열어보자.

```js
// src/main.js
console.log(import.meta);
```

node 명령어로 main.js를 실행시키면 SyntaxError가 발생한다. 이유는, node.js는 기본적으로 CommonJS 환경에서 동작하기 때문이다. import.meta는 ESM 환경에서 동작하기 때문에 이런 에러가 발생하는 것이다.

Node.js에서 import.meta를 사용하는 방법은 2가지인데, 하나는 main.js 파일을 `main.mjs` 로 ESM에서 동작하는 JavaScript 확장자를 붙여주어 node 명령어로 다시 실행시키는 방법이고, 다른 하나는 `dotevn` 라이브러리에서 ESM 환경으로 동작시킬 때 알아본 package.json 파일에 `"type": "module"` 을 명시해주는 방법이다.

```js
// src/main.js
console.log(import.meta);

/**
 * [Object: null prototype] {
 *   dirname: '/Users/hwangsuntae/Documents/dev/play-ground/html-exam/src',
 *   filename: '/Users/hwangsuntae/Documents/dev/play-ground/html-exam/src/main.js',
 *   resolve: [Function: resolve],
 *   url: 'file:///Users/hwangsuntae/Documents/dev/play-ground/html-exam/src/main.js'
 * }
*/
```

위와 같은 결과를 얻었다. dirname, filename, resolve, url 등의 정보가 들어있다. 절대 경로로 내 파일(모듈)의 위치를 가리키고 있다. 그렇다면 브라우저에서는 어떻게 동작하는가?

간단하게 [http-server](https://github.com/http-party/http-server) 를 설치하고 index.html 파일을 브라우저로 서빙해보자.

```sh
$ npm i -D http-server
```

```json
// package.json
{
  // ...
  "type": "module",
  "scripts": {
    "dev": "http-server ./"
  },
  "dependencies": {
    "http-server": "^14.1.1"
  }
}

```

이렇게 dev 명령어를 입력해주고, index.html 파일을 만들어 `npm run dev` 명령어를 실행한다.

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <script type="module" src="./src/main.js"></script>
  </head>
  <body></body>
</html>
```

[browser-import.meta]

resolve와, url 정보가 들어있다. Node.js 환경과는 조금 다르다. dirname, filename이 없다. 그리고 Node.js 환경에서의 `url` 은 모듈(파일)의 절대 경로가 들어가 있고, 브라우저 환경에서의 `url` 은 http server가 서빙하고 있는 주소 자체를 담고 있다.

여기서 재미있는 것을 하나 더 해볼 수 있는데, 바로 쿼리 파라미터로 원하는 값을 넣어줄 수 있다는 것이다.

```html
<script type="module" src="./src/main.js?info=5"></script>
```

이렇게, 쿼리 파라미터 `info` 라는 key에 value를 5를 넣었다. 다시 실행해보면, main.js가 모듈로서 여전히 호출되고

[import.meta.info]

이렇게 url의 모습이 변경된 것을 볼 수 있다. 그러면 이 url 변수를 가공해 info 값을 얻을 수가 있는데,

```js
// src/main.js
const info = new URL(import.meta.url).searchParams.get("info");
console.log(info); // 5
```

이런 식으로 모듈 끼리의 정보도 주고 받을 수 있다. 이는 Node.js 환경에서도 마찬가지이다.

```js
// some.js
export function callImportMeta() {
  return new URL(import.meta.url).searchParams.get("id");
}
```

```js
// main.js
import { callImportMeta } from "./some.js?id=pozafly";
console.log(callImportMeta()); // pozafly
```

메타 정보를 쿼리 파라미터로 주고 받았다.

node.js와 browser 모두 meta 정보에 접근할 수 있다. 근데 vite에서는 왜 dotevn 를 쓰는걸까? env를 단순히 읽어들이기 위해서일까?

vite로 프로젝트를 실행하고 Chrome devTools의 source 탭에 App.tsx가 트랜스파일링 된 파일을 보면

[source-tab]

이렇게 되어 있다. env 파일에 정의한 `VITE_xxx` 변수 및 다른 변수가 브라우저 외부로 노출되어 있다. Vite는 우선 devServer(Node.js 환경)에서 환경 변수를 설정 할 뿐 아니라, 위의 쿼리 파라미터를 이용해 다른 기능에도 응용하고 있다. 이 쿼리 파라미터를 어떻게 사용하고 있을까?

<br/>

## HMR

HMR(Hot Module Replacement)는 번들러 환경에서 개발할 때 주로 사용하는데, IDE의 소스코드를 변경하면 devServer를 새로 껐다키지 않고 브라우저를 새로고침 하지 않아도 모듈을 교체해 변경된 부분을 즉시 볼 수 있게 만들어주는 기능이다.

Vite에서는 HMR 기능에 `import.meta` 를 사용하고 있다. 브라우저에 로드 된 Sources 탭의 소스맵에 breakpoint를 찍어 보면서 따라가보자.

[dev-tools-break-point]

HTML head 태그에 `/@vite/client` 를 불러오도록 명시하고 있다. 따라서 진입점은 `client.ts` 파일이다.

```js
// @vite/client.ts (간추린 버전)
const importMetaUrl = new URL(import.meta.url);
const socketProtocol = importMetaUrl.protocol === "https:" ? "wss" : "ws";
const socketHost = `${importMetaUrl.hostname}:${importMetaUrl.port}`;

let socket = setupWebSocket(socketProtocol, socketHost);

function setupWebSocket(protocol, hostAndPath) {
  const socket = new WebSocket(`${protocol}://${hostAndPath}`, "vite-hmr");
  let isOpened = false;

  // Listen for messages
  socket.addEventListener("message", async ({ data }) => {
    handleMessage(JSON.parse(data));
  });

  return socket;
}
```

`importMetaUrl` 은 import.meta.url을 URL 객체화 한 정보를 가지고 있다.

```
importMetaUrl: URL
  hash: ""
  host: "localhost:5173"
  hostname: "localhost"
  href: "http://localhost:5173/@vite/client"
  origin: "http://localhost:5173"
  password: ""
  pathname: "/@vite/client"
  port: "5173"
  protocol: "http:"
  search: ""
  searchParams: URLSearchParams
    size: 0
  username: ""
```

위에서 살펴봤던 브라우저에 로드 된 `@vite/client` 모듈의 정보가 담겨있다. 쿼리 스트링에 담겨있을 `searchParams` 는 아직은 존재하지 않는다. 다시 소스를 보면, socket 연결을 위한 작업들을 하고 있다. 이 정보를 봤을 때, Vite의 devServer와 브라우저가 소켓 통신을 통해 어떤 정보를 주고 받을 준비를 하는 것을 볼 수 있다.

그리고 socket으로 `message` 이벤트를 감지하고 있는데, 브라우저의 소켓에서 메세지 이벤트를 devServer로 부터 전달 받으로 `handleMessage` 함수를 실행하도록 되어있다. 이를 기억하자.

이제 HTML의 body 하단의 `<script type="module" src="/src/main.tsx"></script>` 코드가 실행되고, 어플리케이션이 실행된다. 렌더링이 종료되어 브라우저에 코드가 실행되고난 후, IDE의 소스코드를 수정하면 재미있는 현상을 볼 수 있다.

[t-query-params]

바로, 소스코드를 수정한 모듈에 쿼리 파라미터 `t` 가 새롭게 붙은 파일이 브라우저로 로드되어 새롭게 대체되는 현상이다.

```js
// @vite/client.ts (간추린 버전)
async function handleMessage(payload) {
  switch (payload.type) {
    // ...
    case "update":
      if (isFirstUpdate) {
        window.location.reload();
        return;
      }
      await Promise.all(
        payload.updates.map(async (update) => {
          if (update.type === "js-update") {
            return hmrClient.fetchUpdate(update);
          }
          // CSS 관련 HMR 소스코드 더 있음.
        })
      );
  }
}
```

socket을 브라우저에 등록하는 과정에서 message 이벤트가 일어나면 실행할 handleMessage 함수를 등록했다. 바로 그 함수가 실행되는데 JavaScript 함수 이므로 `hmrClient.fetchUpdate()` 함수가 실행된다.

hmrClient는 마찬가지로 Vite가 처음 브라우저로 넘어왔을 때 만들어진 인스턴스인데, 이는 아래와 같다.

```js
// @vite/client.ts (간추린 버전)
const hmrClient = new HMRClient(async function importUpdatedModule({
  acceptedPath,
  timestamp,
  explicitImportRequired,
}) {
  const [acceptedPathWithoutQuery, query] = acceptedPath.split(`?`);
  return await import(
    base +
      acceptedPathWithoutQuery.slice(1) +
      `?${explicitImportRequired ? "import&" : ""}t=${timestamp}${query ? `&${query}` : ""}`
  );
});
```

이곳이 핵심이다. `await import()` 를 사용해 어떤 다른 모듈을 동적으로 불러오는 것을 볼 수 있다. `t=` 로 시작하는 쿼리 파라미터가 있으며, t에는 timestamp가 존재한다. dev-tools에서 봤던, `App.tsx?t=xxx` 는 이것이다.

HMRClient class를 보자.

```js
// @vite/hmr.ts (간추린 버전)
export class HMRClient {
  constructor(importUpdatedModule) {
    this.importUpdatedModule = importUpdatedModule
  }

  public async fetchUpdate(update) {
    const { path, acceptedPath } = update;
    const isSelfUpdate = path === acceptedPath;

    if (isSelfUpdate) {
      await this.importUpdatedModule(update);
    }

    return () => {
      for (const { deps, fn } of qualifiedCallbacks) {
        fn(deps.map((dep) => (dep === acceptedPath ? fetchedModule : undefined)));
      }
    };
  }
}
```

`importUpdatedModule` 을 받는다. 이는 HMRClinet class를 생성할 때 받는 함수 자체를 말하는데, 결국 이게 실행되는 것은, `await imort()` 동적 모듈을 가져오는 과정이다.

결국 `@vite/client.ts` 에 있던 socket의 addEventListener에 등록된 `handleMessage` 에서 넘겨 받은 payload 안에 `update` 객체가 들어있고, update 객체에서는 이것이 js인지, css인지 구분하는 값과, timestamp 값, query 등의 값이 들어있다. 즉, 이 정보는 모두 devServer에서 만들어주어 socket으로 브라우저로 전송되어 입력되며 동적 import를 통해 다시 IDE로부터 수정된 파일을 devServer에서 다시 브라우저로 전송하는 것이다.

이를 도식화 해보면 아래와 같다.

[hmr-process]

Vite는 import.meta를 통해 모듈의 정보를 이용해 HMR를 구현했다. url을 통해 모듈의 경로와 메타 데이터를 활용해 socket를 세팅하고, IDE에서 소스코드 변경이 일어나면 모든 모듈을 교체하는 것이 아니라 변경된 코드가 담긴 모듈만 동적 import를 통해 교체하는 것이다.

`import.meta.hot` 을 콘솔에 찍어보면

[import.meta.hot]

이런 정보를 가지고 있고, 그 중 `hotModulesMap` 도 들어있다.

[hotModulesMap]

hotModulesMap은, 현재 소스코드를 Vite가 code splitting을 통해 모듈로 나눠놓은 것을 담고 있다. 이 모듈에서 socket으로 부터 받은 payload의 정보를 통해 모듈을 교체하기 때문에 훨씬 효율적으로 모듈을 교체할 수 있는 것이다.

그래서, 소스코드를 수정할 때마다 새로운 timestamp가 쿼리 파라미터가 브라우저에 넘어오고, 쿼리 파라미터 정보를 import.meta로 읽어 새로운 모듈을 가져와야 한다는 정보를 얻어 devServer에 요청 후 갈아끼울 수 있는 매커니즘을 가질 수 있었다.

<br/>

아래를 참조하자.

- import.meta가 proposal에 올라간 사유: https://www.proposals.es/proposals/import.meta
- How to Access ES Module Metadata using import.meta: https://dmitripavlutin.com/javascript-import-meta/
- https://blog.bitsrc.io/understanding-import-meta-your-key-to-es-module-metadata-e2911b0d3c00
