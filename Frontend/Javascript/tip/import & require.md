# Import & require

> 출처: https://www.daleseo.com/js-module-require/

차이점에 대해 알아보자.

- require: node.js에서 사용되고 있는 CommonJS 키워드.
- import: ES6에서 새롭게 도입된 키워드.

둘 다 모두 하나의 파일에서 다른 파일의 코드를 불러온다는 동일한 목적을 가짐.

```js
const moment = require('moment');
import moment from 'moment';
```

위 코드는 기본적으로 MomentJS 라는 라이브러리를 물러오는 동일한 작업을 하고 있음. CommonJS 방식을 따른 `require` 코드는 다른 변수를 할당하듯 모듈을 불러오는 반면, ES6 방식을 따른 `import` 는 좀 더 명시적으로 모듈을 불러옴.

<br/>

## Require

### CommonJS 모듈 시스템의 필요성

많은 플젝에서 ES6 모듈 시스템을 점점 더 널리 사용하고 있지만, 모든 시스템에서 import 구문을 쓸 수 있는 것은 아님. `<script>` 태그를 사용하는 브라우저 환경에서는 물론이고, NodeJS 에서도 CommonJS 를 기본 모듈 시스템으로 채택하고 있기 때문에, Babel 과 같은 ES6 코드를 변환 해주는 도구를 사용할 수 없는 상황에서는 무조건 `require` 키워드를 사용해야 한다.

### 주의사항

CommonJS 방식으로 모듈을 내보낼 때는 ES6 처럼 명시적으로 선언하는 것이 아니라 특정 변수나 그 변수의 속성으로 내보낼 객체를 세팅해주어야 함. 유사해보이는 `exports`, `module.exports` 변수를 상황에 맞게 잘 사용해야 함.

1. 여러 개의 객체를 내보낼 경우, exports 변수 속성으로 할당.
2. 딱 하나의 객체를 내보낼 경우, module.exports 변수 자체에 할당한다.

### 복수 객체 내보내기 / 불러오기

하나의 JS 모듈 파일에서 여러 개의 객체를 내보내고 불러오는 방법을 알아보자.

---

#### 내보내기

아래는 미국과 캐나다 달러를 상호 변환해주는 JS 코드임. 이 파일엔 3개의 함수가 있음. 아래 2개의 함수만 다른 파일에서 접근할 수 있도록 내보내기 하겠음. `exports` 변수의 속성으로 내보낼 함수들을 세팅함.

- a1.js

```js
const exchangeRate = 0.91;

function roundTwoDecimals(amount) {
  return Math.round(amount * 100) / 100;
}

const canadianToUs = function(canadian) {
  return roundTwoDecimals(canadian * exchangeRate);
}

function usToCanadian(us) {
  return roundTwoDecimals(us / exchangeRate);
}

exports.canadianToUs = canadianToUs;  // 내보내기 1
exports.usToCanadian = usToCanadian;  // 내보내기 2
```

#### 불러오기

위에서 내보낸 객체는 `require` 키워드를 통해 한번에 불러와 변수에 할당할 수 있다면, 그 변수를 통해서 내보낸 객체에 접근할 수 있음.

- b.js

```js
const a = require('./a');

console.log(a.canadianToUs(50)); // 45.5
console.log(a.usToCanadian(30)); // 32.97
```

---

### 단일 객체 내보내기

하나의 JS 모듈 파일에서 단 하나의 객체를 내보내고 불러오는 방법을 알아보자.

---

#### 내보내기

두개 함수를 객체로 묶어서 내보내보자. 내보낼 객체를 `module.exports` 변수에 할당해주면 된다.

- a.js

```js
const exchangeRate = 0.91;

// 안내보냄
function roundTwoDecimals(amount) {
  return Math.round(amount * 100) / 100;
}

// 내보내기
const obj = {};
obj.canadianToUs = function (canadian) {
  return roundTwoDecimals(canadian * exchangeRate);
};
obj.usToCanadian = function (us) {
  return roundTwoDecimals(us / exchangeRate);
};
module.exports = obj;
```

#### 불러오기

위에서 내보낸 하나의 객체는 `require` 키워드를 통해 변수에 할당할 수 있고, 그 변수를 통해 일반 객체에 접근하는 것 처럼 속성에 세팅되어 있는 함수에 접근할 수 있다.

- b.js

```js
const a = require("./a");

console.log(a.canadianToUs(50)); // 45.5
console.log(a.usToCanadian(30)); // 32.97
```

<br/>

## Import

### ES6 모듈 시스템의 이점

좀 더 최신 스펙이다 보니 CommonJS 방식 대비 이점이 많다. `import`, `from`, `export`, `default` 처럼 모듈 관리 전용 키워드를 사용하기 때문에 가독성이 좋다. 비동기 방식으로 작동하고 모듈에서 실제로 쓰이는 부분만 불러오기 때문에 성능과 메모리 부분에서 유리하다.

### 복수 객체 내보내기 / 불러오기

하나의 JS 모듈 파일에서 여러 개의 객체를 내보내고 불러오는 방법을 알아보자.

---

#### 내보내기

CommonJS 에서는 내보낼 복수 객체들을 `exports` 변수의 속성으로 할당하는 방식을 사용했지만, ES6 에서는 import와 export 키워드를 명시적으로 선언함. 이 때 내보내는 변수나 함수의 이름이 그대로 불러낼 때 사용하게 되는 이름을 `Named Exports` 라고 부른다.

- a.js

```js
const exchangeRate = 0.91;

function roundTwoDecimals(amount) {
  return Math.round(amount * 100) / 100;
}

const canadianToUs = function(canadian) {
  return roundTwoDecimals(canadian * exchangeRate);
}

function usToCanadian(us) {
  return roundTwoDecimals(us / exchangeRate);
}
export { usToCanadian };
```

#### 불러오기

불러올 때는 ES6의 Destructuring 문법을 사용해 필요한 객체만 선택적으로 전역에서 사용하거나, 모든 객체에 별명을 붙이고 그 별명을 통해 접근할 수 있음.

- b.js

```js
// Destructuring
import { canadianToUs } from "./currency-functions";
console.log(canadianToUs(50)); // 45.5

// Alias
import * as currency from "./currency-functions";
console.log(currency.usToCanadian(30)); // 32.97
```

---

### 단일 객체 내보내기 / 불러오기

#### 내보내기

CommonJS에서는 내보낼 단일 객체를 `module.exports` 변수에 할당하는 방식을 사용, ES6에서는 `export default` 로 내보냄. 하나의 모듈에서 하나의 객체만 내보내기 때문에 `Default Export` 라고 함.

- a.js

```js
const exchangeRate = 0.91

// 안 내보냄
function roundTwoDecimals(amount) {
  return Math.round(amount * 100) / 100
}

// 내보내기
export default {
  canadianToUs(canadian) {
    return roundTwoDecimals(canadian * exchangeRate);
  },

  usToCanadian: function (us) {
    return roundTwoDecimals(us / exchangeRate);
  },
}
```

#### 불러오기

하나의 객체를 불러올 땐, 간단하게 `import` 키워드를 사용해 아무 이름이나 우너하는 이름을 주고 해당 객체를 통해 속성에 접근하면 됌.

- b.js

```js
import a from "./a";

console.log(currency.canadianToUs(50)); // 45.5
console.log(currency.usToCanadian(30)); // 32.97
```

<br/>

한 가지 주의할 점은 `Babel` 없이 순수하게 Node.js 최신 버전으로 ES 모듈을 사용한다면 `import` 를 사용할 때, `.js` 확장자를 붙여주어야 함.
