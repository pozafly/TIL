# Provider Patterns

> [출처](https://mong-blog.tistory.com/entry/Vue-prop-drilling%EC%9D%84-%EC%98%88%EB%B0%A9%ED%95%98%EB%8A%94-Provider-Pattern-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

## Prop drilling 단점

### 복잡성

- props를 여러 컴포넌트를 거쳐 전달하면, 사용치 않은 데이터가 컴포넌트에 선언되기 때문에 코드 복잡도가 올라간다.
- 특정 컴포넌트에서 props 전달만 하고 사용하지 않는다면, 필요치 않는 코드라고 생각되어 삭제될 여지가 생김.

### 유지 보수 어려움

- props 데이터 변경 시, 중간 컴포넌트에서 이상이 없는지 확인하는 절차가 필요.
- 만약 어떤 컴포넌트는 props 데이터를 전달만 하고 또 다른 컴포넌트는 props 데이터를 재가공해서 전달하고 있다면 props 데이터 변경에 따른 사이드 이펙트 발생

### 디버깅의 어려움

- 데이터가 여러 컴포넌트를 통해 전달되면 문제가 발생한 곳을 파악하기 어려울 수 있음.
- 만약 이슈가 E 컴포넌트에서 발생했다면, 우리는 E 컴포넌트를 먼저 확인한다.
- 하지만, 문제가 발생한 데이터가 A 컴포넌트에서 비롯된 거라면 데이터의 시작점을 찾는 과정이 동반 됨.

<br/>

## props drilling 예방

### 1) vuex, pinia와 같은 전역 데이터 사용?

- pinia, vuex는 데이터를 중앙에서 관리하기에 데이터 변경을 추적하기 쉬움.
- 데이터를 '**중앙**'에서 관리하는 것이 목적이기에 남용시 성능이 저하될 수 있다.
- 데이털르 중앙 관리하면 어떤 도메인에서든 데이터를 변경할 수 있게 되는데, 이 경우 데이터의 범위가 의도와 달리 넓어진다.

### 2) provider pattern

- vue에서 제공하는 `provide`, `inject` 를 사용하는 방법
- 이 패턴의 핵심은 바로 상위 컴포넌트에서 `provide`로 값을 전달하고, 이 값을 필요로 하는 하위 컴포넌트에서 `inject` 를 사용해 값에 접근하는 것이다.

![image](https://github.com/pozafly/TIL/assets/59427983/9528a892-f801-488e-a665-07103a9aa272)

#### provide: 값을 전달한다

- provide는 데이터를 컴포넌트 트리 전체에서 사용할 수 있다.
- 중간 컴포넌트를 거치지 않고 데이터를 전달할 수 있다.
- 두 개의 인수를 받는데, 첫번째는 키, 두 번째는 값.

```js
provide(key, value);
```

- 만약 컴포넌트 트리에 message 키를 가진 값을 전달하고 싶다면, 아래와 같이 사용할 수 있다.

```vue
<script setup>
// ParentComponent.vue
import { provide } from 'vue';

provide('message', 'hello!');
</script>
```

#### inject: 전달한 값에 접근한다.

- 하위 컴포넌트에서 inject에 사용하면 provide한 값에 접근할 수 있다.

```vue
<script setup>
// ChildComponent.vue
import { inject } from 'vue';

const message = inject('message');
console.log(message); // hello!
</script>
```

<br/>

## Provider pattern 더 잘쓰는 법

### 1) provide 키를 심볼로 관리

- string 타입으로 키를 지정해도 되는데, 큰 프로젝트에서는 어떤 키가 있는지 여부와 키가 중복되어 값이 수정되지 않았는지 파악이 어렵다.
- 하지만, 심볼을 쓰면 실수로 값을 덮어쓰는 경우를 막을 수 있음.
- 키의 무결성과 안정성을 보장할 수 있다.
- 심볼키를 지정할 때는 `Object.freeze`와 `Symbol` 두가지를 사용할 수 있다.

#### Object.freeze

- Object.freeze는 객체의 수정을 막는 함수.
- 객체의 값을 수정하거나 삭제하려고 하면 변경되지 않는다.

```js
const obj = {'a': 12, 'b': 13};

Object.freeze(obj);
obj.a = 44;          // {'a': 12, 'b': 13}  - 변경 ❌
obj.c = 11;          // {'a': 12, 'b': 13}  - 추가 ❌
delete obj.a;        // {'a': 12, 'b': 13}  - 삭제 ❌
```

- 적용해보자

```js
// provider-symbol.ts

export const symbols = Object.freeze({
    loginService: 'loginService',
    customerService: 'customerService',
  ..
})
```

```vue
<script lang="ts" setup>
// parentComponent.vue

import { provide } from 'vue';
import { symbols } from '@/provider-symbol';  // symbol 파일을 import해서

provide(symobols.loginService, { /* 어떤_데이터 */ });     // symbol키 사용하기 
</script>
```

```vue
<script lang="ts" setup>
// ChildComponent.vue

import { inject } from 'vue'
import { symbols } from '@/provider-symbol'

inject(symbols.loginService);
</script>
```

#### Symbol로 키 선언

```js
const sym1 = Symbol();
const sym2 = Symbol("foo");
const sym3 = Symbol("foo");
```

- 고유하기 때문에 이름이 같은 심볼이라도 다른 값이기 때문에 고유한 값으로 평가된다. 만약 심볼 값이 같은지 확인하고 싶다면 `Symbol.keyFor()`, `Symbol.for()` 를 사용할 수 있다.

``` js
Symbol("foo") === Symbol("foo"); // false

Symbol.keyFor(Symbol.for("foo")) === "foo"; // true
```

- 적용해보자.

```js
// provider-symbol.ts

export const loginService = Symbol('loginService');
export const customerService = Symbol('customerService');
```

```vue
<script lang="ts" setup>
// ParentComponent.vue

import { provide } from 'vue'
import { loginService } from '@/provider-symbol'  // symbol 파일을 import해서

provide(loginService, { /* 어떤_데이터 */ });       // symbol키 사용하기 
</script>
```

```vue
<script lang="ts" setup>
// ChildComponent.vue

import { inject } from 'vue'
import { loginService } from '@/provider-symbol'

inject(loginService);
</script>
```

#### Object.freeaze, Symbol로 키 선언

- 마지막으로 두 가지를 합쳐 키 값을 유니크 하게 유지하면서, 키의 변경도 막을 수 있다.

```js
// provider-symbol.ts

export const symbols = Object.freeze({
    loginService: Symbol('loginService'),
    customerService: Symbol('customerService'),
  ..
})
```

### 2) provide 로직을 composable로 분리

#### composable이 필요한 순간?

```vue
<script lang="ts" setup>
// parentComponent.vue

import { provide } from 'vue';
import { symbols } from '@/provider-symbol'; 

provide(symobols.loginService, () => { 
  // 이 로직을 provide의 값으로 넘기고 있다.
  const userInfo = reactive({});
  function login(email, pw) { ... }
  function checkEmail(email) { ... }
  function checkPassward(pw) { ... }

  return { userInfo, login, checkEmail, checkPassward };
});   
</script>
```

하지만 만약 데이터 로직이 너무 길어 컴포넌트에서 선언했을 때 로직 구분이 어렵다면?

- 이 때 컴포저블을 사용할 수 있다.
- 컴포저블은 로직을 캡슐화 하기에, 코드의 재사용성과 유지 보수성을 향상시킬 수 있다.
- 유명한 컴포저블 예시는 [vueuse](https://vueuse.org/core/useLocalStorage/#uselocalstorage) 인데, 그 중 코드가 간단한 [useLocalStorage](https://github.com/vueuse/vueuse/blob/c95a02bc46b9571f7a41fb6d7dc1ad03e548a3c0/packages/core/useLocalStorage/index.ts)를 가져와봤다.

```js
// vueuse의 useLocalStorage 내부 코드
export function useLocalStorage(key, initialValue, options: = {}) {
  const { window = defaultWindow } = options;

  return useStorage(key, initialValue, window?.localStorage, options)
}
```

- useLocalStorage 내부는 컴포저블 함수 명을 `useXXX` 형태로 명함. 특정 데이터에 대한 로직을 함수로 묶어두었다.
- 그럼 컴포저블을 `provide`와 어떻게 함께 사용하나?

#### provide, composable 예시

- 우선 provide로 넘기는 로직을 별도 파일로 분리해야 함.

```vue
<script lang="ts" setup>
// parentComponent.vue

import { provide } from 'vue';
import { symbols } from '@/provider-symbol'; 

provide(symobols.loginService, () => { 
  // 아래 로직을 별도 파일로 분리해야함
  const userInfo = reactive({});
  function login(email, pw) { ... }
  function checkEmail(email) { ... }
  function checkPassward(pw) { ... }

  return { userInfo, login, checkEmail, checkPassward };
});   
</script>
```

- 분리한 코드는 로그인 처리 로직이기에 `useLogin` 으로 명명하고 파일 명도 `use-login` 으로 설정한다.

```js
// use-login.ts
// 데이터 로직을 담은 컴포저블

import { reactive, computed } from 'vue';

export function useLogin() {
  const userInfo = reactive({});
  function login(email, pw) { ... }
  function checkEmail(email) { ... }
  function checkPassward(pw) { ... }

  return { userInfo, login, checkEmail, checkPassward };
}
```

- 그 다음, `useLogin`을 `import`하여 `provide`로 넘겨주면 된다!

```vue
<script lang="ts" setup>
// parentComponent.vue

import { provide } from 'vue';
import { symbols } from '@/provider-symbol'; 
import { useLogin } from '@/use-login'; 

provide(symobols.loginService, useLogin);   
</script>
```

### 3) 키 값이 없을 때 예외 처리

- provider pattern을 사용할 때 발생하는 실수 중 하나가 유효하지않은 키에 접근하는 경우다.
- 예방하기 위해 `inject` 에서 예외 처리할 수 있다.

```js
function requireInjection(key, value) {
    const resolved = inject(key, value);
    if (!resolved) {
    throw new Error(`${key} was not provided.`);
  }
    return resolved;
}
```

```vue
<script lang="ts" setup>
// parentComponent.vue

import { provide } from 'vue';
import { symbols } from '@/provider-symbol'; 
import { useLogin } from '@/use-login'; 
import { requireInjection } from '@/utils'

requireInjection(symobols.loginService, useLogin);   
</script>
```

- 혹은 `inject` 키가 없다면 `provide`를 강제하는 방법도 있다.

```javascript
function requireInjection(key, value) {
  const resolved = inject(key, value);
  return resolved ? resolved : provide(key, value);
}
```



---

- 이번 시간에는 Vue에서 `prop drilling`을 예방하기 위해 `Provider Pattern`을 사용하는 방법에 대해 알아보았다. `Provider Pattern`은 컴포넌트 간에 데이터를 효율적으로 공유하기 위한 패턴이다.
- `prop drilling`은 여러 개의 하위 컴포넌트를 거쳐 데이터를 전달해야 하는 번거로움이 있었다. 이로 인해 유지보수성과 디버깅이 어려워지는 문제가 발생했는데, `Provider Pattern`을 이용해 데이터를 필요로 하는 컴포넌트로 직접 주입시킬 수 있어 이 문제를 예방할 수 있었다.
- 만약 프로젝트의 크기가 크다면 `Provider Pattern`을 사용해 `prop drilling`을 예방해보면 어떨까?