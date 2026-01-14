# CommonJS는 사라지지 않는다

> [출처](https://velog.io/@surim014/commonJS-is-not-going-away)

[최근](https://bun.sh/blog/bun-v0.6.5) Bun의 [릴리즈](https://bun.sh/blog/bun-v0.6.10) [노트](https://bun.sh/blog/bun-v0.6.12)에서 CommonJS 지원이 언급된 것을 보고 놀라신 분들도 계실 것임. 결국 CommonJS는 레거시 모듈 시스템이고 자바스크립트 미래는 ESM이라고 여겨지기 때문.

그럼에도 미래 지향적 차세대 런타임인 Bun이 CommonJS 지원 개선에 그토록 많은 노력을 기울인 이유는?

![[assets/images/b1e1947d84e8de20cb7269ea33ef7870_MD5.png]]

CommonJS는 계속 사용될 것이다. CJS, ESM 간 상호 운용성에 대한 개발자 경험 문제를 해결할 수 있다고 생각한다.

<br/>

## 상황 설명

어플리케이션을 분할하는 것은 좋은 방법이다. 다른 파일에 있는 코드를 참조할 수 있는 방법이 필요하다.

CJS 모듈 형식은 2009년에 개발되어 Node.js에 의해 대중화 되었다. `exports` 라는 특수 변수에 속성을 할당할 수 있다. 그런 다음, 다른 파일은 `require` 라는 특수 함수를 사용해 파일을 '요청'함으로써 `exports` 객체의 속성을 참조할 수 있다.

```js
// a.js
const b = require("./b.js");

b.sayHi(); // "hi" 출력
```

```js
// b.js
exports.sayHi = () => {
  console.log("hi");
};
```

파일이 `require` 될 때, 파일이 **실행**되고 `exports` 객체의 속성을 importer가 사용할 수 있게 된다. CommonJS는 서버 측 JavaScript(사실 원래 이름은 ServerJS였다)를 위해 설계되었으며, 모든 파일을 로컬 파일 시스템에서 사용할 수 있어야 함을 전제로 한다.

이는 바로 CJS가 **동기식**이라는 의미다. `require()` 를 가져온 파일을 읽고 실행한 다음 importer에게 다시 제어권을 넘겨주는 '블로킹' 작업으로 개념화할 수 있다.

ECMAScript 모듈은 2015년에 ES6로 도입됨. ESM은 `export` 키워드 사용, 내보내기를 선언한다. `import` 키워드는 다른 파일에서 가져오는데 사용된다. `exports/require`과 달리 `import/export`는 모두 파일의 **최상위 수준**에서만 발생할 수 있다.

```js
// a.js
import { sayHi } from "./b.js"

sayHi(); // "hi" 출력
```

```javascript
// b.js
export const sayHi = () => {
  console.log("hi");
};
```

ESM은 브라우저에서 작동하도록 설계되었기 때문에 파일이 네트워크를 통해 로드될 것을 예상하고 만들어짐. ESM 모듈이 **비동기식**이라는 의미다. ESM이 주어지면 브라우저는 파일을 실행하지 않고도 import/export 하는 내용을 확인할 수 있다. 일반적으로 코드가 실행되기 전 전체 모듈 그래프가 해결된다. (잠재적으로 여러 번의 왕복 네트워크 요청이 포함될 수 있다)

<br/>

## CommonJS의 경우

### CJS가 더 빠르게 시작된다

ESM은 대규모 어플리케이션에서 속도가 더 느리다. require와 달리 문(statement)을 사용할 때 전체 모듈 그래프를 로드해야 하거나 각 표현식의 import를 기다려야 한다. 예를 들어, 함수에서 사용하기 위해 패키지를 지연 로드하는 경우, 코드에서 **반드시** 프로미스를 반환해야 한다(추가적인 마이크로틱과 오버헤드가 발생할 수 있다)

> 번역자 주: microticks는 JavaScript 실행 환경에서 비동기 작업을 처리하는 작은 작업 단위를 뜻한다.

```js
async function transpileEsm(code) {
  const { transform } = await import("@babel/core");
  // ... Promise를 반환해야 한다.
}

function transpileCjs(code) {
  const { transform } = require("@babel/core");
  // ... return은 동기식 이다.
}
```

ESM은 설계상 속도가 느리다. import를 export에 바인딩하려면 두 번의 패스가 필요하다. 전체 모듈 그래프가 구문 분석되고 분석된 다음 코드가 평가된다. 이과정은 여러 단계로 나뉜다. 이것이 바로 ESM에서 '라이브 바인딩'을 가능하게 하는 요소다.

두 개의 간단 파일을 살펴보자.

```js
// babel.cjs
require("@babel/core")
```

```javascript
// babel.mjs
import "@babel/core";
```

바벨은 수많은 파일로 구성된 패키지이므로, 이 두 파일의 런타임을 비교함으로써 모듈 해석과 관련된 성능 비용을 적절하게 평가할 수 있다. 결과는 다음과 같다.

![[assets/images/707b71f67125149e252361f5a3e1fa72_MD5.png]]

> Bun을 사용하면 CommonJS로 Babel을 로드하는 속도가 ES Module을 사용하는 것보다 약 2.4배 빠르다.

85ms의 차이가 있다. 서버리스 콜드 스타트의 맥락에서 이는 엄청난 차이다. Node.js의 경우, 1.8배 (~60ms) 차이였다.

### 증분 로딩

CJS는 조건에 따라 파일을 `require()` 하거나, 동적으로 구성된 경로/지정자를 `require()` 하거나, 함수 본문에 `require()` 를 사용할 수 있는 등 동적 모듈 로딩이 가능하다. 이런 유연성은 플러그인 시스템이나 사용자 상호 작용을 기반으로 한 특정 컴포넌트를 지연로드하는 경우와 같이 동적 로딩이 필요한 시나리오에서 유용하게 사용할 수 있다.

ESM은 유사한 속성을 가진 동적 `import()` 함수를 제공한다. 어떤 의미에서 이 함수의 존재는 CJS의 동적 접근 방식이 유용하고 개발자들에게 가치를 인정받고 있다는 사실을 입증하는 증거다.

### 이미 사용중이다

npm에 게시된 수백만 개의 모듈이 이미 CJS를 사용하고 있다. 이 중 상당수는 (a) 더이상 활발하게 유지보수되지 않으면서, (b) 기존 프로젝트에 매우 중요한 모듈이다. 모든 패키지가 ESM을 사용할 것으로 기대할 수 있는 시점은 결코 오지 않을 것이다. CJS를 지원하지 않는 런타임 똔느 프레임워크는 엄청난 가치를 놓치고 있다.

<br/>

## Bun의 CommonJS

Bun v0.6.5 부터 Bun 런타임은 기본적으로 CommonJS를 구현한다. 이전에는 Bun이 CommonJS 파일을 특수한 '동기식 ESM' 형식으로 트랜스파일 했다.

### ESM에서 CommonJS 가져오기

ESM에서 CJS 모듈을 `import` 하거나 `require` 할 수 있다.

```js
import { stuff } from "./my-commonjs.cjs";
import Stuff from "./my-commonjs.cjs";
const myStuff = require("./my-commonjs.cjs");
```

최근에 Bun은 [`__esModule` 어노테이션](https://github.com/oven-sh/bun/pull/3393)에 대한 지원도 추가했다.

```js
// module.js
exports.__esModule = true;
exports.default = 5;
exports.foo = "foo";
```

이는 CJS 모듈이 (`package.json`의 `"type": "module"` 과 함께) `exports.default` 를 default export로 해설해야 함을 나타내는 사실상의 매커니즘이다. CJS 모듈에 `__esModule` 이 설정되어 있는 경우, default `import` (`import a from "./a.js"`) 를 통해 `exports.default` 속성을 가져오게 된다. 어노테이션이 없으면 default import는 전체 `exports` 객체를 가져온다.

어노테이션을 사용하는 경우, 다음과 같다.

```js
// __esModule: true를 사용하는 경우
import mod, { foo } from "./module.js";
mod; // 5
foo; // "foo"
```

어노테이션을 사용하지 않는 경우, 다음과 같다.

```js
// __esModule를 사용하지 않는 경우
import mod, { foo } from "./module.js";
mod; // { default: 5 }
mod.default; // 5
foo; // "foo"
```

이는 CommonJS 모듈이 `exports.default` 를 default export로 해설해야 함을 나타내는 사실상 표준 방식이다.

<br/>

## 요약

CJS는 여전히 존재한다. 또한 존재해야 하는 실질적인 이유도 있다. Bun의 ESM을 좋아하지만 실용성도 놓칠 수 없다. CJS는 과거 시대의 유물이 아니며, Bun은 CJS를 일류 시민으로 취급한다.
