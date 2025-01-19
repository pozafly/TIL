# Vite, webpack의 treeShaking

> [출처1](https://bepyan.github.io/blog/2023/bundlers), [출처2](https://so-so.dev/web/tree-shaking-module-system/), [출처3](https://ui.toast.com/posts/ko_20220127)

우선엔 vite에서는, devServer에서 esBuild를 사용하고, 프로덕션 번들링은 rollup.js를 사용한다.

webpack과 rollup에 대한 차이점을 공부하며 느꼈던 부분을 작성한다.

## ESM vs CJS

webpack은, commonJS를 사용하고, rollup은 ESM을 사용한다. commonjs는 treeshaking에 불리하고, ESM은 treeshaking에 유리하다. 이유는 정적 분석에 유리한지 어려운지의 여부에 따라 달라짐.

CJS는 동적으로 모듈을 해석할 수 있다.

```js
// CJS
var foo = 'foo';
var lib = require(`lib/${foo}`);
lib.someFunc(); // property lookup
```

require 문에 foo라는 변수를 대입함으로서 어떤 모듈을 가져올지 **동적**으로 판단이 가능하기 때문이다. 반면, ESM은 정적으로 모듈을 해석하는 방법을 사용한다. 즉, 동적으로 어떤 모듈을 가져올지 할당이 불가능하다.

```js
// ESM
import foo from `lib/${foo}`;  // SyntaxError

function foo () {
  export default 'bar' // SyntaxError
}
foo()
```

이렇게 최상위에만, import, export가 가능하며, if 문 내부에 모듈을 가져올 수 없다. 따라서, 정적인 모듈만 가져올 수 있는데, 아래와 같이 동적 import는 또다른 의미를 가진다.

```js
// ESM
let {hi, bye} = await import('./say.js');
```

위 구조는 **비동기**로 모듈을 가져오기 때문에 코드 스플리팅이 가능하다. 어쨌든 위의 동적, 정적의 의미와는 조금 다른 것임.

CJS는 위와 같이 동적으로 가져오기 때문에, CJS를 사용하는 webpack에서 번들링을 진행할 시, 사용하지 않는 코드도 모두 번들에 포함된다. 왜냐하면 언제든 런타임에서 모듈을 가져올 가능성이 있기 때문이다. 즉, treeshaking이 되지 않는다. (어렵다)

반면 rollup에서는 ESM을 사용하기 때문에 어느 코드가 어느 상황에서 쓰이며, 어느 코드가 사용하지 않는 코드 조각인지 정적으로 판단 가능하기 때문에 treeshaking을 했을 경우 잘 가져오며, 필요 없는 부분은 가져오지 않게 된다.

그런 이유로, webpack은 사용하지 않는 코드라 할지라도 가져오므로 더 안정적이라고 할 수 있다. 반면 rollup은 사용하지 않는 코드를 배제하기 때문에 treeshaking 성능은 좋지만, 안정성이 떨어진다고 볼 수 있다.

ESM은 **정적인 구조**를 가지고 있다는 점이 가장 중요하다.

<br/>

## TreeShaking

treeShaking 개념은 rollup에서 가장 먼저 나왔다. 정적으로 분석할 수 있는 ESM에 유리했기 때문이다. treeShaking은 '정적 분석이 가능한 구조에 대해 더 잘 지원할 수 있다.'

webpack과 rollup은 treeShaking 기능을 바라보는 관점이 조금 다르다.

- webpack: 사용하지 않는 코드를 제거하는 관점
- rollup: 사용하는 코드만 가져오는 관점

![image](https://github.com/pozafly/TIL/assets/59427983/69fc9c6b-3b4a-4294-aa32-804eaef3e3a4)

왼쪽이 webpack, 오른쪽이 rollup이다. rollup은 live code inclusion(필요한 코드만 쌓아가는 과정). 즉, 어떤 모듈이 필요한지 평가하는 과정이다.

webpack5에서는 terser-webpack-plugin이 기본으로 제공되기 때문에 죽은 코드를 제거하는 과정이 수행된다.

이런 관점 차이 때문에 webpack은, 모듈을 묶을 때, 변수명 충돌이 발생하지 않도록 모듈들을 함수로 감싼다. 즉시 실행 함수로, 클로저를 만들어 변수명을 격리시키는 작업. 그리고, rollup은 모듈을 호이스팅해 한번에 평가한다.

<br/>

## ESBuild

webpack, rollup은 JavaScript로 번들링한다. 하지만, ESBuild는 내부적으로 go로 작성되어 있다. 10~100배 빠른 번들링.

하지만, 설정이 webpack, rollup처럼 유연하지 못하고 code-splitting 및 CSS 관련 처리가 미흡하다. 그리고 es5 이하 문법을 아직 100% 지원하지는 않는다.

<br/>

## Vite

- ESM 기반 강력한 개발서버
- esbuild로 파일을 통합하고, rollup을 통해 번들링

로컬 서버를 구동하는데 vite는 코드를 2가지로 분류 하여 번들링한다.

- Dependencies: node_modules에 있는 종속성이다.
  - node_modules에 있는 파일을 번들링 하려면 시간이 오래걸렸었음.
  - vite는 종속성을 `사전 번들링`을 하는데 ESBuild를 사용한다.
- Source code: JSX, CSS 등 컴파일이 필요하고, 수정이 잦은 코드다.
  - Native ESM을 이용해 코드를 제공한다.
  - 조건부 동적 import 이후 실제 화면에서 사용되는 경우에만 처리한다.

> 📌 여기서 Dependencies의 사전 번들링(pre-bundling)이란?
>
> 프리 번들링이란, vite에서는 처음 실행하여 사이트를 로컬로 로드하기 전에 프로젝트 **종속성을 미리 번들로 제공**하는 것을 말한다.
>
> 종속성은 변하지 않는다. vite 는 모든 파일을 ESM으로 제공하기 때문에 CJS, UMD로 제공되는 종속성을 ESM으로 변환해야 한다. esbuild를 통해 변하지 않는 종속성을 빠르게 제공하기 위해 빌드 후 로컬 파일에 저장해 두고, 이를 적시에 서빙함.
>
> <img width="324" alt="스크린샷 2023-11-04 오후 1 08 45" src="https://github.com/pozafly/TIL/assets/59427983/8c47a6a7-934d-4727-a7d5-55712b34187b">
>
> 이미지와 같이, node_modules 폴더에 `.vite` 폴더 하위에 종속성이 사전 번들링 된 결과다. 단, 프로덕션 빌드를 할 경우 rollup/plugin-commonjs가 사용된다.

그럼 HMR처럼 자주 변경되는 Source code는 어떻게 갱신 시킬까? webpack과 같은 경우는 다시 모든 파일을 번들링 했다. 느림.

vite도 HMR를 지원하는데, 번들러가 아닌 ESM을 이용한다.

<br/>

### 프로덕션 모드

개발 모드에서는 ESM을 사용했지만, 추가 네트워크 통신이 필요한 것은 비효율적이다. (HTTP2를 사용하더라도.) 그리고 작은 번들을 얻기 위해 tree Shaking, 지연 로딩, code-spliiting이 필요하므로 rollup을 사용한다.

### 비교

webpack, rollup, parcel은 소스코드와 node_modules 폴더의 전체 코드 베이스를 번들로 묶고, 빌드 프로세스(babel, ts, postCSS)를 통해 번들된 코드를 브라우저에 전달한다. 하지만, 캐싱 및 최적화 작업을 하더라도 대규모 크롤링을 하는 개발 서버를 느리게 만듦.

Snowpack, vite 및 wmr 개발 서버는 위의 모델을 따르지 않음. 브라우저가 삽입 구문을 찾고 HTTP로 해당 모듈을 요청할 때까지 기다림. http 요청이 오면 로컬 서버는 요청 모듈과 모듈의 가져오기 트리에 있는 모든 잎 노드에 변환을 적용한 후 브라우저에 제공함. 이러면 모든 번들을 새로 평가하지 않기 때문에 작업 속도가 훨씬 빨라디낟.
