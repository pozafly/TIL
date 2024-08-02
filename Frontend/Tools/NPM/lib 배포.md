# lib 배포

```json
// package.json
{
  "name": "npm-lib-sample",
  "version": "0.0.1",
  "description": "",
  "main": "src/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT"
}
```

package.json에 이렇게 적은 뒤, `src/index.js` 에 내용을 적는다.

## CommonJS

```js
module.exports = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => a / b,
};
```

commonJS 방식으로 일단 해봄.

https://www.npmjs.com/login?next=/login/cli/b81c04b2-c1e7-4807-a0b0-f1843d1fd138

이곳에 회원가입.

```sh
$ npm login
```

로그인 한다.

```sh
$ npm publish
```

하면 로그인 한, id로 배포가 된다.

<br/>

## ESM

package.json에 module을 변경해서 ESM 방식으로 해도 된다.

```json
// package.json
{
  "name": "npm-lib-sample",
  "version": "0.0.1",
  "description": "",
  "type": "module", // here
  "main": "src/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT"
}
```

type module을 적어주면 그 때부터 ESM 방식으로 동작한다.

그럼 기존의 cjs로 작성된 index.js를 `index.cjs` 으로 이름을 변경하자. 왜냐하면 type: module 을 붙이는 순간 js 파일들은 모두 esm 방식으로 동작하고, `.cjs` 를 반드시 붙여주어야만 CommonJS 방식으로 동작하기 때문이다.

```js
// index.cjs
module.exports = {
  ver: 'cjs',
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => a / b,
};
```

```js
// index.esm.js
export const ver = 'esm';
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => a / b;
```

이렇게 만들어주자.

```json
// package.json
{
  ...
  "type": "module",
  "main": "src/index.cjs",
  "module": "src/index.esm.js",
  "exports": {
    ".": {
      "require": "./src/index.cjs",
      "import": "./src/index.esm.js"
    }
  },
  ...
}
```

이러면 2가지 cjs, mjs 모두를 지원하는 라이브러리가 되었다.

- `main` : require 하거나 import 할 때 사용할 파일을 지정한다. CommonJS 형식인 index.cjs가 기본 진입점임
- `module` : ESM 형식으로 패키지를 사용할 때 기본 진입점. import 할 때 이 파일이 로드된다.
- `exports` : 모듈을 외부에서 사용할 때 진입점을 더 세밀하게 제어할 수 있다. ESM과 CJS 형식을 모두 지원할 때 유용하다.
  - `requrie` : CJS 형식으로 불러올 때임.
  - `import` : ESM 형식으로 불러올 때임.

<br/>

## 사용해보기

프로젝트를 세팅하고 publish 된 라이브러리를 다운 받아 사용해보자.

```sh
$ pnpm i [라이브러리명]
```

```js
// index.js
import { ver } from 'npm-lib-sample';

console.log(ver); // esm
```

```js
// index.cjs
const { ver } = require('npm-lib-sample');

console.log(ver); // cjs
```

두 형태 모두 다 잘 된다.

<br/>

## type 지원

index.d.ts 파일을 만들어 타입을 지정해보자.

```js
// index.d.ts

export declare const ver: string;
export declare const add: (a: number, b: number) => number;
export declare const subtract: (a: number, b: number) => number;
export declare const multiply: (a: number, b: number) => number;
export declare const divide: (a: number, b: number) => number;
```

```json
// package.json
{
	...
	"types": "src/index.d.ts",
  "exports": {
    ".": {
      "types": "./src/index.d.ts" // 추가
    }
  },
	...
}
```

이렇게 하면

<img width="420" alt="image" src="https://github.com/user-attachments/assets/610cd89d-ee9b-4cbb-b2be-723f6f3d4338">

이렇게 index.d.ts에 정의된 타입을 가져오는 것을 볼 수 있다.

<br/>

## Tooling (tsdx, vite)

이렇게 CJS, ESM 방식으로 둘 다 만들었지만, 실제 라이브러리는 CJS, ESM 둘 중 하나의 방식으로 만들고, vite와 같은 번들러로 변환하여 두 모듈 시스템 버전으로 내보낸다.

[tsdx](https://tsdx.io/) 는 명령어로 타입스크립트 라이브러리를 손쉽게 만들어준다.

```sh
$ npx tsdx create mylib
```

<img width="397" alt="image" src="https://github.com/user-attachments/assets/8f046c90-f503-4c08-bdb2-216af564b387">

고르라고 함. `react-with-storybook` 을 해보자. 그러면 storybook을 통한 typescript template를 만들어준다.

<img width="223" alt="image" src="https://github.com/user-attachments/assets/3d102f2f-0c3f-4247-b271-2815193eca70">

하지만 관리된지는 오래되었음. 다른 대안으로는 vite가 있음.

[vite - library-mode](https://vitejs.dev/guide/build#library-mode) 에 가보면 앱이 아닌 라이브러리를 만드는 것으로 사용할 수 있음.

<br/>

## monorepo

라이브러리 조각 외 다른 필요한 파일이 많이 있다. 예를 들면 데모 앱 같은 것(npm 패키지를 설치해서 실제로 사용한 github에 올린 소스 등). 웹 사이트를 만들어야 하는 경우도 있음. 그리고 라이브러리를 소개하는 web site도 필요할 것임.

이렇게 연관있는 패키지들을 묶는데 필요한 것이 monorepo 임.

monorepo는 하나의 repository에 여러 개의 프로젝트를 모아두는 것을 말한다.

만들어보자. 기존 파일들을 모두 library 폴더를 만들고 넣어두고, root에 package.json을 새로 하나 만든다.

<img width="202" alt="image" src="https://github.com/user-attachments/assets/3830707f-54fd-445f-9c62-0fa42434a108">

```json
// package.json
{
  "name": "npm-lib-sample-root",
  "private": "true",
  "workspaces": [
    "library",
    "apps/*"
  ]
}
```

root에 생성한 곳에는 

- `private: true` : 이 전체 패키지는 npm에 배포되어서는 안된다는 의미다. 하나 하나 각각의 패키지는 배포될 수 있어도, 이건 root이기 때문에 private로 하는 것임.
- workspaces : 워크스페이스 설정이다. library 코드와 demo 코드가 이 패키지를 묶는다.

이 구조에서 demo의 package.json에

```js
// apps/demo/package.json
{
  "name": "demo",
  "version": "0.0.1",
  "scripts": {
    "test": "echo test"
  }
}
```

`npm run test` 를 실행시키고 싶다면, `$ cd apps/demo` 로 들어가서 `npm run test` 를 해야하지만,

```sh
$ yarn workspace demo test
```

를 해도 이제는 된다. pnpm은 아래와 같음.

```sh
$ pnpm --filter demo test
```

그럼 demo니까, 라이브러리 제작한걸 다운받고 실행시켜보자.

```sh
$ pnpm --filter i -D npm-lib-sample
```

```json
{
  "name": "demo",
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "node index.js",
    "test": "echo test"
  },
  "devDependencies": {
    "npm-lib-sample": "^0.0.6"
  }
}
```

이렇게 `dev` scripts를 넣어주고, `$ pnpm --filter demo dev` 명령어로 실행할 수 있음.

```js
// apps/demo/index.js
import { ver, add } from 'npm-lib-sample';

console.log(ver);
console.log(add(1, 2));
```

잘 실행되는걸 볼 수 있음.

웹 사이트는 [docusaurus](https://docusaurus.io/ko/), [vitepress](https://vitepress.dev/) 같은 것으로 만들어서, 배포할 수도 있음.


