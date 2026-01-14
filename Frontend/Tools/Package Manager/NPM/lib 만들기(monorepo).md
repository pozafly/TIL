# Lib 배포2

## 배포 환경 잡기

root에 `$ npm init` 으로 package.json을 만들고 아래를 넣자.
```json
// package.json
{
  ...
  "workspaces": [
    "packages/*"
  ],
  "private": true,
  ...
}
```
workspaces는 packages로 실제 라이브러리가 들어갈 것이고, root 프로젝트이기 때문에 private true를 넣어준다.

packages 디렉토리를 만들고 터미널로 packages 폴더로 이동 후

`$ yarn create vite` 로 프로젝트를 만든다. project name은 `library` 다.

![[assets/images/82dd173b42f78ec25aa01ce582914bfc_MD5.png]]

이런 구조가 되었을 것이다. library의 package.json 파일은 아래와 같다.
```json
{
  "name": "@pozafly/mini-query",
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "typescript": "^5.2.2",
    "vite": "^5.3.4"
  }
}
```
`"name": "@pozafly/mini-query",` 이렇게 넣어주자. `@pozafly/` 는 `scoped name` 이라고 이야기 한다.

📌 scope 된 라이브러리는 private 하게 배포되는 것이 default다.

예를 들어 회사에서 사용하는 npm에 publish 된 라이브러리 `@equal/` 같은 경우 외부에 노출이 되면 안되기 때문이다.

하지만, 우리는 이걸 public 하게 배포하고 싶으니
```sh
$ npm publish --access public
```
이라고 배포해주어야 한다.

[vite - library-mode](https://vitejs.dev/guide/build#library-mode) 를 보고 vite.config.js을 만들어 넣어주자.
```js
import { resolve } from 'path';
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/main.ts'),
      name: 'MiniQuery',
      fileName: 'mini-query',
    },
  },
});
```
rollupOptions는 필요 없으니 지워주자.

라이브러리를 작성 (src/main.ts 파일 작성) 후
```js
// src/main.ts
export const name = 'pozafly';
```
터미널이 root로 오게 한 후 `$ yarn workspace @pozafly/mini-query build` 로 빌드를 한 번 해보자. yarn install도 하고.

만약 에러가 난다면 yarn install을 하지 않아서 임. 근데 root에서 매번 이 명령어를 치기 귀찮으니 package.json에 build 스크립트를 넣어두자.
```json
"scripts": {
  "build": "yarn workspace @pozafly/mini-query build",
  ...
},
```
![[assets/images/5bf135b0101198e03db171bc110f5638_MD5.png]]

이렇게 dist 폴더가 생성되었다.
```js
// mini-query.js

const o = "pozafly";
export {
  o as name
};
```
생성된 mini-query.js는 이렇게 생겼음.
```json
// packages/package.json
{
  ...
  "files": ["dist"],
  "main": "./dist/mini-query.umd.cjs",
  "module": "./dist/mini-query.js",
  "exports": {
    ".": {
      "import": "./dist/mini-query.js",
      "require": "./dist/mini-query.umd.cjs"
    }
  },
  ...
}
```
이제, packages 안의 package.json 파일에 `files` 옵션을 넣어주어야 함. files 배열 안에 들어가는 폴더가 npm에 배포 될 것이기 때문임.

그리고, main, module, exports 등을 통해 진입점을 잡아주도록 하자.

<br/>

## Test
```sh
$ yarn workspace @pozafly/mini-query add vitest -D
```
min-query repo에 vitest를 install 해준다. `src/__test__/main.test.ts` 를 만들고,
```ts
import { describe, expect, it } from 'vitest';
import { $ } from '../main';

describe('MiniQuery', () => {
  it('does nothing', () => {
    expect(true).toBe(true);
  });

  it('returns length correctly', () => {
    const div = document.createElement('div');
    div.innerHTML = `
      <button class="btn" type="button">button 1</button>
      <button class="btn" type="button">button 2</button>
      <button class="btn" type="button">button 3</button>
      <button class="btn" type="button">button 4</button>
    `;

    expect($('.btn').length()).toBe(4);
  });
});
```
이렇게 넣어주자. 그런데, 처음엔 실패할 것이다. node 환경은 `document` 를 모르기 때문이다.

vitest에 jsdom을 넣어주어야 한다. node 환경에서 가상 DOM을 넣는 과정임.
```js
import { resolve } from 'path';
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
  },
  build: {
    ...
  },
});
```
이렇게 넣어주고, `$ yarn workspace @pozafly/mini-query add json -D` 로 jsdom을 설치해주자. 잘 된다.

<br/>

## Demo 만들기

root에 apps 폴더를 만들고 yarn create vite로 example을 만들자. root의 package.json에 workspaces에 `apps/*` 를 넣어 워크스페이스를 잡아주자.

`$ yarn workspace example dev` 를 통해 실행이 잘 되는지 보자.

### 라이브러리 dependency 추가

apps/example/package.json에 `dependency` 를 추가해주자.
```json
// apps/example/package.json
{
  ...
	"dependencies": {
    "@pozafly/mini-query": "0.0.1"
  },
}
```
yarn workspace를 사용할 때, root 레벨에서 `yarn install` 하면, 하위에 있는 모든 패키지의 디펜던시들을 모두 한번에 설치하게 된다. 즉, 각 패키지 내에 node_modules에 깔리는게 아니라 📌 root에 패키지들을 설치하게 된다.

example에서 패키지를 사용할 수 있었던 이유는, `yarn workspace` 가 가져와주기 때문이다.

테스트 해보자.

apps/example/src/main.js에
```js
import { $ } from '@pozafly/mini-query';

console.log('Hello', $);
```
이렇게 넣고 `yarn workspace example dev` 명령어로 실행시켜보자.

![[assets/images/be6119fe35f006de273da86a331cd1cf_MD5.png]]

에러가 남. 이건 library를 빌드하지 않아서 그렇슴.

`yarn workspace @pozafly/mini-query build` 명령어로 빌드하면 잘 나오는 걸 볼 수 있음.

click메서드를 만들고 다시 실행해보자.
```js
// packages/library/src/__test__/main.test.ts

describe('click()', () => {
  it('attaches click event listener correctly', () => {
    const div = document.createElement('div');
    div.innerHTML = `
      <button class="btn" type="button">button 1</button>
      <button class="btn" type="button">button 2</button>
      <button class="btn" type="button">button 3</button>
      <button class="btn" type="button">button 4</button>
    `;
    const handler = vi.fn();
    $('.btn').click(handler);

    (div.querySelectorAll('.btn')[0] as HTMLElement).click();

    expect(handler).toBeCalledTimes(1);
  });
});
```
이렇게 했다. 실패임. 왜?

> ⭐️ 실수 하지 않도록 하는 법
>
> `$('.btn').click(handler);` 여기에 container div를 넣어주지 않아서임. 이걸 어떻게 하면 방지할 수 있을까?
>
> ```js
> import { $ as miniQuery } from '../main';
> 
> const $ = (selector: string, container: Element) => {
>   if (!container) {
>     throw new Error('specify container in test cases.');
>   }
>   return miniQuery(selector, container);
> };
> ```
>
> 이렇게 원래의 `$` 를 miniQuery로 ailas를 준 후, `$` 를 래핑 한다. 그리고 container가 없을 경우 Error를 발생시키는 걸로 이제 test 환경에서는 무조건 container를 넣도록 강제해서 에러를 방지할 수 있다.

<br/>

## Lib typescript 개발의 불편함

위의 경우를 생각해보면, 3가지 과정이 있다.

- lib 소스코드 작성
- vitest로 로직 테스트
- vite dev server로 build 된 결과물 브라우저에서 확인

근데, 브라우저에서 라이브러리 테스트를 할 경우, 매번 build를 돌려주어야 page refresh 되면서 라이브러리 소스코드 변경이 가능하다.

이렇게 하지 않으려면 js로 개발하면 된다. vite는 ts를 js로 빌드해주는데, dist 폴더를 매번 바라보고 있어야 하기 때문이다. 이 경우, workspace를 src를 바라보게 하고, js로 개발하면 된다.

그래서 ts 진영에서는 라이브러리 개발은 js로 하고, ts를 사용하는 곳에서는 main.d.ts 파일로 (타입 데피니션 파일) 타입을 지정해주는 방식으로 사용하곤 한다.

<br/>

## 문서 만들기

[docusaurus](https://docusaurus.io/ko/), [vitepress](https://vitepress.dev/), [nextra](https://nextra.site/) 같은걸로 만들 수 있음. vitepress를 써보자.

apps/docs 디렉토리를 만들고 터미널 경로 이동한다.

`$ npx vitepress init` 후, vitepress install 하고, dev로 켜보면 잘 나온다.

### 배포

vercel에 docs를 배포하면 된다.
