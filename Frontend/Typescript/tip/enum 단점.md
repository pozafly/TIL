# enum 단점

> - 출처1 : https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking
> - 출처2 : https://techblog.woowahan.com/9804/

## TypeScript에서 enum을 사용하면 Tree-shaking이 되지 않는다

```ts
export enum MOBILE_OS {
  IOS,
  ANDROID,
}

// 문자열을 할당한 경우
export enum MOBILE_OS2 {
  IOS = "iOS",
  ANDROID = "Android",
}
```

위 코드를 트랜스파일하면 아래와 같은 JavaScript 코드가 된다.

```js
export var MOBILE_OS;
(function (MOBILE_OS) {
    MOBILE_OS[MOBILE_OS["IOS"] = 0] = "IOS";
    MOBILE_OS[MOBILE_OS["ANDROID"] = 1] = "ANDROID";
})(MOBILE_OS || (MOBILE_OS = {}));

// 문자열을 할당한 경우
export var MOBILE_OS2;
(function (MOBILE_OS2) {
    MOBILE_OS2["IOS"] = "iOS";
    MOBILE_OS2["ANDROID"] = "Android";
})(MOBILE_OS2 || (MOBILE_OS2 = {}));
```

JavaScript에 존재하지 않는 것을 구현하기 위해 TypeScript 컴파일러는 `IIFE`를 포함한 코드를 생성한다. 그런데 Rollup과 같은 번들러는 IIFE를 '사용하지 않는 코드'라고 판단할 수 없어 Tree-shaking이 되지 않는다. 결국 MOBILE_OS 를 import하고 실제로는 사용하지 않더라도 최종 번들에는 포함되는 것이다. 단, 이는 rollup을 사용할 때 발생한다.

2023/5 현재는 vite에서 트리쉐이킹이 된다.

<br/>

## enum 대체제

### const enum

```ts
const enum MOBILE_OS = {
  IOS: "iOS",
  ANDROID: "Android",
}

type MOBILE_OS = typeof MOBILE_OS[keyof typeof MOBILE_OS]; // 'iOS' | 'Android'
```

트랜스 파일링 했을 경우 코드가 아예 나오지 않음.

### const assertion

```ts
const MOBILE_OS = {
  IOS: "iOS",
  ANDROID: "Android",
} as const;

type MOBILE_OS = typeof MOBILE_OS[keyof typeof MOBILE_OS]; // 'iOS' | 'Android'
```

사용하면 위와 같이 유니언 타입을 사용해 묶어주어야 한다.

`as const` 키워드는 TypeScript에서 할당을 금지시킨다. [링크](https://stackoverflow.com/questions/66993264/what-does-the-as-const-mean-in-typescript-and-what-is-its-use-case)

```ts
const args = [8, 5]; // 타입 추론으로 number[] 타입이 된다.
const angle = Math.atan2(...args); // error! Expected 2 arguments, but got 0 or more.
```

atans2 메서드는 반드시 2개의 매개변수를 가져야 한다. 따라서 spread 문법을 사용하면 에러가 발생한다. 왜냐햐면 args에 string 값을 추가로 넣을 수도 있고 number를 변환 시킬 수도 있기 때문에.

이 때 as const 키워드를 사용해 상수로 만들어줄 수 있다.

```ts
const args = [8, 5] as const; // 타입 추론으로 readonly [8, 5] 가 된다.
const angle = Math.atan2(...args); // okay
```

즉 읽기 전용 객체로 만드는 것이다.

어쨌든 처음 봤던 MOBILE_OS as const를 트랜스파일링 하면 아래와 같이 그대로 나오는 것을 볼 수 있다.

```js
const MOBILE_OS = {
    IOS: "iOS",
    ANDROID: "Android",
};
```

TypeScript 코드에서는 MOBILE_OS 타입을 정의한 이점을 그대로 누리면서, JavaScript로 트랜스파일해도 IIFE가 생성되지 않기 때문에 Tree-shaking을 할 수 있다.

---

1. const enum
   - 장점
     1. tree-shaking 됨.
     2. js로 변환되고 나면, 매우 적은 용량을 차지.
     3. key 값에 의미 있는 값을 부여할 수 있다.
     4. enum을 쓰는 쪽에서는 오타를 낼 확률이 0%가 된다. 무조건 enum 값으로 써야 하기 때문.(number value 사용하지 않을 때)
   - 단점
     1. Object.keys, Object.entries를 이용해서 값들에 대해 순회를 할 수가 없다.
     2. –isolatedModules 옵션을 사용하면, 사용 불가.
     3. ts 패키지를 만들고, 이를 다른 곳에서 사용한다면, 이를 타입용으로 사용할 수가 없다.
     4. cra, next에서 사용 불가.
     5. babel 설정도 추가로 필요
2. const assertion
   - 장점
     1. tree-shaking 됨.
     2. enum에 비해서 js로 변환되고 나면, 적은 용량을 차지.
     3. 값과 타입 모두 사용 가능하기 때문에, Object.keys, Object.entries를 이용해서 값들에 대해 순회가능.
     4. key 값에 의미 있는 값을 부여할 수 있음.
   - 단점
     1. 이것을 위한 별도의 union type을 만들어야 해서, 성가심.
     2. 원래의 의도가 enum과는 다르기 때문에, 쓰기가 약간은 망설여짐.

---

지금은 vite에서 트리쉐이킹이 되고,

```ts
enum CountryCode5 {
  Argentina = 1,
  China = 2,
  Japan = 3,
}

const country51: CountryCode5 = CountryCode5.China
const country52: CountryCode5 = 2
const country53: CountryCode5 = 10    // no error
```

number 형태로 enum을 사용하더라도 예전엔 없는 number를 넣어도 에러가 나지 않았지만 TypeScript 5버전인 지금은 에러가 발생한다. 즉, 올바르게 잡힌 것이다.



















