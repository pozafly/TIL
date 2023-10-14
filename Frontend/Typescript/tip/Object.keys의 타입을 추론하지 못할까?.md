
# Object.keys의 타입을 추론하지 못할까?

> [출처](https://medium.com/@yujso66/%EB%B2%88%EC%97%AD-%EC%99%9C-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-object-keys%EC%9D%98-%ED%83%80%EC%9E%85%EC%9D%84-%EC%A0%81%EC%A0%88%ED%95%98%EA%B2%8C-%EC%B6%94%EB%A1%A0%ED%95%98%EC%A7%80-%EB%AA%BB%ED%95%A0%EA%B9%8C%EC%9A%94-477253b1aafa)

![image](https://github.com/pozafly/TIL/assets/59427983/df226129-283b-446f-a2a5-af12523b7a59)

`options` 키를 사용해 `options`에 접근하려 하는데, 왜 타입스트립트는 자동으로 알아채지 못할까?

`Object.keys(options)`를 `(keyof typeof options)[]` 로 캐스팅하면 이 문제를 비교적 쉽게 우회할 수 있다.

```ts
const keys = Object.keys(options) as (keyof typeof options)[];
keys.forEach((key) => {
  if (options[key] == null) {
    throw new Error(`Missing option ${key}`);
  }
});
```

근데 왜 처음부터 이게 문제가 되는 것일까?

`Object.keys` 의 타입 정의를 살펴보면 다음과 같은 내용을 확인할 수 있다.

```ts
// typescript/lib/lib.es5.d.ts
interface Object {
  keys(o: object): string[];
}
```

타입 정의는 매우 간단하다. 매개 변수가 `object` 이고, `string[]`을 반환하는 함수다. 이 메서드가 제네릭 매개변수 `T` 를 받아들이고 `(keyof T)[]`를 반환하도록 만드는 것은 매우 간단하다.

```ts
class Object {
  keys<T extends object>(o: T): (keyof T)[];
}
```

`Object.keys`가 이렇게 정의되었다면 타입 에러가 발생하지 않았을 것이다.

`Object.keys`를 이렇게 정의하는 것은 당연한 것처럼 보이지만 타입스크립트에서 그렇게 하지 않은 데에는 그럴만한 이유가 있다. 타입스크립트 [구조적 타입 시스템](https://en.wikipedia.org/wiki/Structural_type_system)과 관련이 있다.





<br/>

## TypeScript 구조적 타이핑

프로퍼티가 누락되었거나 잘못된 타입일 때 에러를 표시한다.

![image](https://github.com/pozafly/TIL/assets/59427983/99628e21-5a95-4e45-a5b8-e44d9d3e3ab6)

그러나 타입스크립트는 추가 프로퍼티가 포함되어 있어도 에러를 표시하지 않는다.

```ts
function saveUser(user: { name: string; age: number }) {}

const user = { name: 'Alex', age: 25, city: 'Reykjavík' };
saveUser(user); // 타입 에러가 아님
```

이는 구조적 타입 시스템에서 의도된 동작이다. 타입 `A`가 `B`의 슈퍼셋인 경우 (`A`는 `B`의 모든 프로퍼티를 포함) 타입 `A`를 `B`에 할당할 수 있다.

그러나 `A`가 `B`의 적절한 슈퍼셋인 경우(즉, `A`가 `B` 보다 더 많은 프로퍼티를 가지고 있는 경우), 다음과 같다.

- `A`는 `B`에 할당 가능하지만
- `B`는 `A`에 할당할 수 없다.

> 참고 : 프로퍼티 측면에서 슈퍼셋이어야 하는 것 외에도 프로퍼티의 타입도 중요하다.

![image](https://github.com/pozafly/TIL/assets/59427983/bcd328ae-0b04-44c2-a28d-e052391e6dd6)

핵심 요점은 `T` 타입 객체가 있을 때, 해당 객체에 대해 아는 것은 `T`의 프로퍼티를 적어도 하나 포함하고 있다는 것 뿐이다.

정확히 `T`에 대한 프로퍼티만 가지고 있는지 여부는 **알 수 없기** 때문에 `Object.keys` 가 그대로 입력된다. 예를 들어보자.

<br/>

## Object.keys의 안전하지 않은 사용

새로운 사용자를 생성하는 웹 서비스의 엔드 포인트를 만든다고 가정하자. 다음과 같은 기존 `User` 인터페이스가 있다.

```ts
interface User {
  name: string;
  password: string;
}
```

사용자를 DB에 저장하기 전 `User` 객체가 유효한지 확인해야 한다.

- `name`은 비어있지 않아야 함.
- `password`는 6자 이상이어야 함.

`validators` 객체를 만들자.

```ts
const validators = {
  name: (name: string) => (name.length < 1 ? 'Name must not be empty' : ''),
  password: (password: string) =>
    password.length < 6 ? 'Password must be at least 6 characters' : '',
};
```

그런 다음 유효성 검사기를 통해 `User` 객체를 실행하는 `validateUser` 함수를 만들자.

```ts
function validateUser(user: User) {
  // 유효성 검사기를 통해 User 객체 전달
}
```

`user`에 있는 각 프로퍼티의 유효성을 검사하고 싶으므로 `Object.keys`를 사용하여 `user`에 있는 프로퍼티를 순회하며 값을 가져올 수 있다.

```ts
function validateUser(user: User) {
  let error = '';
  for (const key of Object.keys(user)) {
    const validate = validators[key];
    error ||= validate(user[key]);
  }
  return error;
}
```

> *참고: 이 코드 블록에는 현재 숨기고 있는 타입 에러가 있습니다. 이 에러는 나중에 해결할 것임.*

이 접근 방식의 문제점은 `user` 객체에 `validators` 에 존재하지 않는 프로퍼티가 포함될 수 있다는 것이다.

```ts
interface User {
  name: string;
  password: string;
}

function validateUser(user: User) {}

const user = {
  name: 'Alex',
  password: '1234',
  email: 'alex@example.com',
};
validateUser(user); // OK!
```

`User`가 `email` 프로퍼티를 지정하지 않더라도 구조적 타이핑을 통해 불필요한 프로퍼티를 제공할 수 있으므로 이는 타입 에러가 아니다. 런타임에 `email` 프로퍼티를 제공할 수 있으므로 이는 타입 에러가 아니다.

런 타임에 `email` 프로퍼티로 인해 `valiator`가 `undefined`가 될 것이고 호출될 때 오류가 발생하게 된다.

![image](https://github.com/pozafly/TIL/assets/59427983/c4217d76-2fcc-493e-9446-b9e1e6b7832b)

다행히도 이 코드가 실행되기 전 TypeScript 에러가 발생함.

![image](https://github.com/pozafly/TIL/assets/59427983/dabc32b6-26a7-4c70-852b-54e17fd2f113)

이제 `Object.keys` 가 현재의 타입으로 정의된 이유에 대한 답을 얻었다. 타입 시슽템이 인식하지 못하는 프로퍼티를 객체에 포함할 수 있다는 점을 받아들어야 한다.

<br/>

## 구조적 타이핑 활용하기

구조적 타이핑은 많은 유연성을 제공한다. 인터페이스가 필요한 프로퍼티를 정확하게 선언할 수 있다.

`KeyboardEvent`를 파싱하고 트리거할 단축키를 반환하는 함수를 작성했다고 가정해보자.

```ts
function getKeyboardShortcut(e: KeyboardEvent) {
  if (e.key === 's' && e.metaKey) {
    return 'save';
  }
  if (e.key === 'o' && e.metaKey) {
    return 'open';
  }
  return null;
}
```

코드가 예상대로 작동하는지 확인을 위해 몇 가지 단위 테스트를 작성해보자.

```ts
expect(getKeyboardShortcut({ key: 's', metaKey: true })).toEqual('save');
expect(getKeyboardShortcut({ key: 'o', metaKey: true })).toEqual('open');
expect(getKeyboardShortcut({ key: 's', metaKey: false })).toEqual(null);
```

괜찮아보이지만 타입 에러가 발생한다.

![image](https://github.com/pozafly/TIL/assets/59427983/6b1cd7ec-97e3-4286-a52c-36627f4722db)

37개의 추가 프로퍼티를 모두 지정하는 것은 매우 번거롭기 때문에 불가능하다.

`KeyboardEvent`에 인수를 캐스팅하면 이 문제를 해결할 수 있다.

```ts
getKeyboardShortcut({ key: 's', metaKey: true } as KeyboardEvent);
```

그러나 이 경우 발생할 수 있는 다른 타입 에러가 가려질 수 있다.

대신 이벤트에서 필요한 프로퍼티만 선언하도록 `getKeyboardShortcut`을 업데이트 할 수 있다.

```ts
interface KeyboardShortcutEvent {
  key: string;
  metaKey: boolean;
}

function getKeyboardShortcut(e: KeyboardShortcutEvent) {}
```

이제 테스트 코드는 이 최소한의 인터페이스만 충족하면 되므로 에러가 발생하지 않는다. 또한 함수가 글로벌 `KeyboardEvent` 타입에 덜 종속되어 더 많은 컨텍스트에서 사용할 수 있다. 훨씬 더 유연해졌다.

이는 구조적 타이핑 덕분에 가능하다. `KeyboardEvent`에 37개의 관련 없는 프로퍼티가 있더라도 `KeyboaardEvent`는 슈퍼셋이기 때문에 `KeyboardShortcutEvent`에 할당할 수 있다.

```ts
window.addEventListener('keydown', (e: KeyboardEvent) => {
  const shortcut = getKeyboardShortcut(e); // 정상입니다!
  if (shortcut) {
    execShortcut(shortcut);
  }
});
```

이 아이디어는 Evan Martin의 글인 [인터페이스는 일반적으로 사용자에게 속합니다](https://neugierig.org/software/blog/2019/11/interface-pattern.html)에서 살펴볼 수 있다. 꼭 읽어보시기를! 이 글은 타입스크립트 코드를 작성하고 생각하는 방식을 바꾸어 놓았다.

이 게시물은 [해커 뉴스에서 많은 흥미로운 토론](https://news.ycombinator.com/item?id=36457557)을 불러 일으켰다.

