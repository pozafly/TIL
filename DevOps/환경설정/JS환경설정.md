# JavaScript 프로젝트 시작하기

## fnm (Fast Node Manager)

```
brew install fnm
```

`~/.bashrc` 또는 `~/.zshrc`에 다음을 추가한다.

```
## setting fnm
export PATH="/Users/pozafly/Library/Application Support/fnm:$PATH"
eval "$(fnm env --use-on-cd)"
```

현재 터미널에서 바로 사용하고 싶다면 위 명령을 그대로 입력한다.

## Node.js 설치

설치 가능한 버전 확인.

```
fnm ls-remote
```

LTS(Long Term Support) 버전을 설치하고 기본으로 사용

```
fnm install --lts
fnm use lts-latest
fnm default $(fnm current)
```

설치된 상태 확인.

```
fnm list

fnm current
```

## NPM 업그레이드

```
npm install -g npm
```

## 프로젝트 폴더 생성

프로젝트 이름을 `my-project`라고 했을 때 다음과 같이 폴더를 만들고 사용할 Node.js 버전을 잡아준다.

```
mkdir my-project
cd my-project
fnm use default
echo "$(fnm current)" > .nvmrc
```

나중에 시스템에 설치된 Node.js 버전과 프로젝트에서 사용하는 Node.js 버전이 다른 상황이 오더라도 `fnm use` 명령을 통해 프로젝트에서 사용하고 있는 버전을 쉽게 사용할 수 있다.

또는 `.nvmrc` 파일을 확인함으로써 어떤 버전으로 개발했는지 알 수 있다.

```
cat .nvmrc
```

## 프로젝트 초기화

다음 명령을 실행하고 질문에 답함으로써 `package.json` 파일을 자동으로 생성한다.

```
npm init
```

귀찮으면 질문에 대해 그냥 엔터만 계속 눌러도 되는데, `-y` 플래그를 사용하면 질문 자체를 안 하게 할 수도 있다.

```
npm init -y
```

<br/>

<br/>

## ESLint

좋은 코딩 스타일을 위해 [ESLint](https://eslint.org/)를 설치해 사용한다.

```
npm install --save-dev eslint
```

다음 명령을 통해 ESLint 설정 파일(`.eslintrc.js`)을 자동으로 생성한다.

```
npx eslint --init
```

몇 가지 질문이 나오는데 프로젝트에 따라 다르게 선택.

```text
? How would you like to configure ESLint?  (Use arrow keys)
❯ Answer questions about your style

? Which version of ECMAScript do you use?  (Use arrow keys)
❯ ES2017

? Are you using ES6 modules? (y/N)
N

? Where will your code run? (Press <space> to select, <a> to toggle all, <i> to invert selection)
 ◯ Browser
❯◉ Node

? Do you use JSX? (y/N)
N

? What style of indentation do you use? (Use arrow keys)
❯ Spaces

? What quotes do you use for strings? (Use arrow keys)
❯ Single

? What line endings do you use? (Use arrow keys)
❯ Unix

? Do you require semicolons? (Y/n)
Y

? What format do you want your config file to be in? (Use arrow keys)
❯ JavaScript
```

`.eslintrc.js` 파일을 열어 `rules`를 수정, 추가한다. [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) 같은 널리 알려진 스타일 가이드를 사용하고 싶다면 간단히 [eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb) 확장을 설치해서 사용하면 된다. 나는 이렇게 씀.

```
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: ['eslint:recommended', 'eslint-config-airbnb'],
  overrides: [],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  rules: {
    // 화살표 함수 매개변수 1개일 때 괄호 빼줌.
    'arrow-parens': ['error', 'as-needed'],
  },
};

```

`.eslintrc.js` 파일 자체를 ESLint 설정에 맞추고 싶다면 다음을 실행한다.

```
npx eslint --fix --no-ignore .eslintrc.js
```

이 설정 파일은 [Visual Studio Code](https://code.visualstudio.com/).

간단하게 테스트를 하기 위해 `index.js` 파일을 작성한다.

```
var a=1
b  =  [
 1
  ,2
]
console . log( b.map(i=>i+a) )
```

고쳐야 할 부분을 찾는다.

```
npx eslint .
```

다음 명령을 실행하면 코드를 검사하고 자동으로 고칠 수 있는 부분을 고쳐주고, 그래도 남아있는 문제는 화면에 보여준다.

```
npx eslint --fix .
```

캐싱할 수 있다.

```js
npx eslint --fix --cache .
```

`package.json`의 `scripts` 항목에 `lint`를 추가하면 이 작업을 편하게 할 수 있다.

```
  "scripts": {
    "lint": "eslint --fix --cache .
  },
```

`npm run`으로 ESLint를 실행할 수 있다.

```
npm run lint
```

아까 테스트하기 위해 만든 `index.js` 파일의 문제가 모두 해결되지 않아서 `npm ERR! code ELIFECYCLE` 에러 메시지가 나오는 걸 볼 수 있다. 간단히 고쳐보자.

```
const { log: print } = console;

var a = 1;
const b = [
  1,
  2,
];

print(b.map(i => i + a));
```

<br/>

<br/>

## Prettier

```
npm i -D prettier
```

eslint와 충돌나지 않게 config도 깔아주자.

```js
npm i -D eslint-config-prettier
```

.prettierrc.js

```js
module.exports = {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  semi: true,
  singleQuote: true,
  trailingComma: 'all',
  bracketSpacing: true,
  // 화살표 함수 매개변수 1개일 때 괄호 빼줌.
  arrowParens: 'avoid',
};
```

<br/>

<br/>

## Husky

git hook을 사용해 prettier, eslint 설정을 통과하지 않으면 commit 과 push가 안되도록 해주는 설정

```shell
npm i -D husky 

(처음 husky 세팅하는 사람만 실행 필요)
npx hushy install
```

`npx hushy install` : husky에 등록된 실제 .git에 적용시키기 위한 스크립트.

그리고, package.json의 "scripts"에 아래를 등록하자. 이후에 clone 받아서 사용하는 사람들은 npm install 후 자동으로 `husky install` 될 수 있도록 하는 설정.

```json
{
	"scripts": {
    "postinstall": "husky install"
  }
}
```

이제 pre-commit, pre-push hook을 설정하자.

```
npx husky add .husky/pre-commit "npm run format"
npx husky add .husky/pre-push "npm run lint"
```

이제 에러를 만나면 commit, push가 안된다.



<br/>

<br/>

## Jest 설치

자동화된 테스트 코드를 작성하고 활용하기 위해 Jest를 설치해서 쓸 수 있다.

```
npm install --save-dev jest
```

`sum.test.js` 파일을 만들어서 확인해 보자.

```
test('sum', () => {
  expect(sum(1, 2)).toBe(3);
});
```

`jest`를 실행하면 `*.test.js` 파일을 모두 실행한다.

```
npx jest
```

테스트를 간단히 통과시키자.

```
const sum = (a, b) => a + b;

test('sum', () => {
  expect(sum(1, 2)).toBe(3);
});
```

만약 파일을 계속 감시하고 있다가 수정될 때마다 자동으로 테스트가 실행되게 하려면 `watchAll` 플래그를 사용하면 된다. 그 상태에서 테스트 전체를 다시 실행하려면 `a`나 `Enter` 키를 누르면 되고, 중단하려면 `q`를 누르면 된다.

```
npx jest --watchAll
```

추가적인 설정이 필요하면 `jest.config.js` 파일을 작성하면 된다. https://jestjs.io/docs/en/configuration 문서 참고.

```
module.exports = {
  verbose: true,
};
```

ESLint를 실행하면 `test`나 `expect` 같은 게 정의되지 않았다는 에러가 뜨는데, `.eslintrc.js` 파일의 `env`에 `jest`를 추가하면 된다.

```
  'env': {
    'es6': true,
    'node': true,
    // Jest 사용
    'jest': true,
  },
```

마찬가지로 `package.json`을 수정해서 `npm`으로 테스트를 실행할 수 있다.

```
  "scripts": {
    "test": "jest  # <- 기존의 에러 종료를 Jest 실행으로 변경",
    "lint": "eslint --fix ."
  },
```

`test`는 기본 명령이라 `run` 없이 실행 가능하다.

```
npm test
```