# Proxy

> [출처](https://leetrue-log.vercel.app/leetrue-proxy-immer)

ES6에서 정의한 글로벌 생성자. Proxy 뜻은 '대리인'이라는 뜻.

> `Proxy` 객체를 사용하면 원래 Object 대신 사용할 수 있는 객체를 만들지만, 이 객체의 속성 가져오기, 설정 및 정의와 같은 기본 객체 작업을 재정의할 수 있다. 프록시 객체는 일반적으로 속성 액세스를 기록하고, 입력의 유효성을 검사하고, 형식을 지정하거나, 삭제하는 데 사용된다.

```js
var target = {}, handler = {};
var proxy = new Proxy(target, handler);
```

이 생성자는 `target` 객체와 `handler` 객체의 인자를 받고, 위와 같은 모습을 가지게 된다.

> Proxy를 생성하는 두 개의 매개변수
>
> - target : Proxy할 원본 객체
> - handler : 가로채는 작업과 가로채는 작업을 재정의하는 방법을 정의한는 객체

proxy 객체는 모든 내부 메서드 호출을 `target` 객체로 전달한다. 즉, `proxy[[Enumerate]]()` 메서드를 직접 호출한다면, `target.[[Eknumerate]]()` 메서드가 그 결과를 반환한다는 것이다.

```javascript
var target = {}, handler = {};
var proxy = new Proxy(target, handler);

proxy.color = 'yellow';
```

위와 같이 `proxy.[[Set]]()` 메서드가 호출될 코드를 아래와 같이 실행시켰을 때 자동적으로 `target.[[Set]]()` 메서드가 호출된다. 그래서 `target` 객체에 새로운 속성이 만들어진다.

![image](https://github.com/pozafly/TIL/assets/59427983/da713fb8-8094-464d-9438-d0e97b0f13c0)

target 객체에 color 속성이 들어있다. 나머지 내부 메서드들도 마찬가지로 동작한다. `proxy` 객체는 대부분의 경우에 `target` 객체인 것처럼 동작한다.

하지만 그렇다고 `proxy` 는 `target` 과 같지는 않다. `proxy !== target // true` 그렇기 때문에 `target` 이 성공하는 타임 체크에 `proxy` 는 실패하곤 한다. 예를 들어 `proxy` 의 `target` 이 DOMElement라면, proxy는 사실 Element가 아니다. 그렇기 때문에 `document.body.appendChild(proxy)` 와 같은 코드는 타입에러를 뱉는다.

<br/>

## Proxy Handler

proxy와 target을 연결 시켰으니, handler 객체를 살펴보자. handler 객체는 이것의 메서스들을 통해 proxy 객체의 모든 내부 메서드들을 오버라이드 할 수 있다.

예를 들면, 객체의 속성에 값을 할당시키는 모든 시도를 가로채고 싶을 때, `handler.set()` 메서드를 정의하면 된다.

```js
var target = {};
var handler = {
  set: function (target, key, value, receiver) {
    throw new Error("Please don't set properties on this object");
  }
};

var proxy = new Proxy(target, handler);
```

위 예시를 보면, `handler` 객체에 `set` 메서드를 정의했다. 그럼 객체의 속성에 값을 할당시켜보면?

<img width="483" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/0e8b419a-5ee6-4de9-b916-8b2df31de497">

set 메서드가 시도를 가로채고 실행되었다.

정리하면, `proxy` 는 `target` 의 대리인 역할을 한다. 그리고 `handler` 는 이것이 가지고 있는 메서드들을 통해 `proxy` 객체가 가진 내부 메서드들의 동작을 가로채 본인이 동작한다. 위 예제에서 proxy의 color 속성을 할당하려고 하자, 동작을 가로채고 `handler` 의 `set` 메서드가 실행 되었다.

> `handler` 객체로 오버라이딩이 가능한 메서드 목록은 아래 MDN 문서에 14개의 메서드들이 작성되어있다. 이 메서드들은 각각이 ES6 가 정의하는 14개의 내부 메서드들에 해당한다.
>
> https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#Methods_of_the_handler_object

<br/>

## 예제

```js
const target = {};
const proxy = new Proxy(target, {
  set(target, key, value, receiver) {
    /*
    	- target: new Proxy의 target 객체
    	- key: propery key
    	- value: propery value
    	- receiver: get의 receiver와 유사하게 동작하는 객체
    */
  }
});

proxy.color = 'red';
```

아래 코드처럼 작성할 수 있다. `proxy.test` 에 무언가 할당하는 코드들을 작성했는데 이 때 할당하려고 하면 인터셉트가 되고, 할당하려고 했던 값들을 `set` 의 `value` 로 전달되어 활용된다.

```js
const target = {};
const proxy = new Proxy(target, {
  set(target, key, value, receiver) {
    if(typeof value !== 'string') {
      console.log('Error:: 문자열을 입력해주세요.');
      return false;
    }
    if (value.length < 3) {
      console.log('Error:: 3자 이상 입력해주세요.');
      return false;
    }
    console.log(`key: ${key}, value: ${value}`);
  }
});

proxy.color = 35; // Error:: 문자열을 입력해주세요.
proxy.color = 'dd'; // Error:: 3자 이상 입력해주세요.
proxy.color = 'abcdef'; // key: color, value: abcdef
```

<br/>

## Object.defineProperty와 차이점

[링크](https://javascript.plainenglish.io/how-is-proxy-better-than-object-defineproperty-why-vue3-started-using-proxy-5353ee54aceb), [링크2](https://borstch.com/blog/proxies-vs-objectdefineproperty-when-to-use-which/)

Vue 2.0에 사용된 `Object.defineProperty`도 객체를 추적할 수 있다. JavaScript 의 오래된 기능으로 개발자가 객체에 대한 새로운 속성을 정의하거나 기존 속성을 수정할 수 있다. getter / setter 함수를 추가할 수 있음.

하지만 한계가 있다. 객체의 **최상위 속성**에서만 작동한다는 점이다. 중첩 객체에 새 속성을 추가하면 Vue에서 추척하지 않는다. 이 제한은 `Vue.set` 하니면 `Vue.delete`를 사용해야 한다는 점이다. 중복된 속성을 추가하거나 제거하는 메소드다.

<br/>

## immer

[도움](https://hmos.dev/deep-dive-to-immer#deep-dive-to-immer)

```js
const baseState = [
  {
    name: 'st',
    age: 10,
  },
  {
    name: 'yg',
    age: 12,
  },
];
```

baseState를 immutable 하게 변경하는 법을 작성해보면 아래와 같다. 복사를 우선적으로 하고 업데이트 하는 과정을 진행하는 식.

```js
const nextState = baseState.slice();

nextState[1] = {
  ...nextState[1],
  done: true,
};

nextState.push({ name: 'gg' });
```

만약 immer를 사용하면 불변성이 자동으로 보장된다.

```js
import produce from 'immer';

const nextState = produce(baseState, draft => {
  draft[1].age = 17;
  draft.push({ name: 'qr', age: 12 });
});
```

immer는 아래와 같은 동작을 함.

- original data와 copy data 2가지 객체를 관리해 원본을 보존하고 copy data만 변경한다.
- 변경한 객체는 modified flag를 켜서 root tree 에서 leaf tree 까지 순회할 수 있도록 한다.
- 변경이 완료된 뒤 modified flag를 이용해 새 객체와 기존 객체를 합성하는 과정을 진행한다.

### produce

```javascript
const immer = new Immer();
export const produce = immer.produce;
export default produce;
```

`produce` 함수는 `Immer` 클래스의 메서드다. 클래스를 보자.

```javascript
export class Immer {
  /**
  - base: 기존 객체
  - recipe: 객체를 어떻게 변경시킬지 결정하는 함수
  */
  produce: (base, recipe) => {
    let result;
    const scope = enterScope(this);
    const proxy = createProxy(this, base, undefined);
    
    result = recipe(proxy);
    return processResult(result, scope);
  }
};
```

