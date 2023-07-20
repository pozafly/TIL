# ESM 모듈 다시 내보내기

> [출처](https://ko.javascript.info/import-export#ref-93)

`export ... from ...` 문법을 사용하면 가져온 개체를 즉시 '다시 내보내기(re-export)' 할 수 있다. 이름을 바꿔서 다시 내보낼 수 있는 것이다.

```js
export {sayHi} from './say.js'; // sayHi를 다시 내보내기 함

export {default as User} from './user.js'; // default export를 다시 내보내기 함
```

다시 내보내기가 왜 필요한 건지 의문이 든다.

NPM을 통해 외부에 공개할 '패키지(package)'를 만들고 있다고 가정하자. 이 패키지는 수 많은 모듈로 구성되어 있는데, 몇몇 모듈은 외부에 공개할 기능을, 몇몇 모듈은 이러한 모듈을 도와주는 '헬퍼' 역할을 담당하고 있다.

```
auth/
    index.js
    user.js
    helpers.js
    tests/
        login.js
    providers/
        github.js
        facebook.js
        ...
```

진입점 역할을 하는 '주요 파일'인 `auth/index.js`를 통해 외부에 노출시키면 이 패키지를 사용하는 개발자들은 아래와 같은 코드로 기능을 사용할 것이다.

```js
import {login, logout} from 'auth/index.js'
```

이 때 만든 패키지를 사용하는 외부 개발자가 패키지 안의 파일들을 뒤져 내부 구조를 건드리게 하면 안된다. 그려려면 공개할 것만, `auth/index.js`에 넣어 내보내기 하고 나머지는 숨기는 것이 좋겠다.

이 때 내보낼 기능을 패키지 전반에 분산하여 구현 후, `auth/index.js`에서 이 기능들을 가져오고 이를 다시 내보내면 원하는 바를 어느 정도 달성할 수 있다.

```js
// 📁 auth/index.js

// login과 logout을 가지고 온 후 바로 내보냅니다.
import {login, logout} from './helpers.js';
export {login, logout};

// User를 가져온 후 바로 내보냅니다.
import User from './user.js';
export {User};
...
```

이제 외부 개발자들은 `import {login} from "auth/index.js"`로 우리가 만든 패키지를 이용할 수 있다.

`export ... from ...`는 위와 같이 개체를 가지고 온 후 바로 내보낼 때 쓸 수 있는 문법이다. 아래 예시는 위 예시와 동일하게 동작합니다.

```js
// 📁 auth/index.js
// login과 logout을 가지고 온 후 바로 내보냅니다.
export {login, logout} from './helpers.js';

// User 가져온 후 바로 내보냅니다.
export {default as User} from './user.js';
...
```

<br/>

## default export 다시 내보내기

기본 내보내기를 다시 내보낼 때는 주의 점이 있다.

```js
// 📁 user.js
export default class User {
  // ...
}
```

1. `User`를 `export User from './user.js'` 로 다시 내보내기 할 때 문법 에러가 발생한다. export default를 다시 내보내려면 위 예시 처럼 `export {default as User}`를 사용해야 한다.

2. `export * from './user.js'`를 사용해 모든 걸 한 번에 다시 내보내면 default export는 무시되고, named export만 다시 보내진다.

   두 가지를 동시에 다시 내보내고 싶다면 두 문을 동시에 사용해야 한다.

   ```js
   export * from './user.js'; // named export를 다시 내보내기
   export {default} from './user.js'; // default export를 다시 내보내기
   ```

default export 를 다시 내보낼 땐 이런 특이 상황도 인지하고 있다가 처리해주어야 하므로 몇몇 개발자들은 default export를 다시 내보내는 것을 선호하지 않는다.
