# 효과적으로 상수 관리하기(as const)

> [출처](https://blog.toycrane.xyz/typescript%EC%97%90%EC%84%9C-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%83%81%EC%88%98-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0-e926db079f9)

## 리터럴 타입

TypeScript에서는 포괄적인 타입(string, number, ...) 외에도 정확한 값을 설정하는 것이 가능하다.

```ts
let title: "typescript";
title = 'javascript';
```

위와 같이 typescript 문자열을 title에 타입으로 부여하면 `title`에 `typescript`가 아닌 다른 값을 입력 시, 컴파일 에러가 발생하게 된다. 즉, string, number 등 포괄적인 의미를 담고 있는 `primitive type` 뿐 아니라 "typescript" 문자 같은 정확한 값을 선언 할 수 있다.

<br/>

## 타입 추론

모든 변수마다 타입을 일일이 적어주는 것은 번거롭다. 따라서 `TypeScript 3.4` 에서 부터는 일일이 타입을 선언하지 않고도 컴파일러가 자동으로 타입을 추론하는 기능을 제공하는데 이에 대해서 알아보도록 하자.

```ts
let title = 'typescript';
```

위 코드는 똑똑하게 title의 타입을 `string으로 추론`한다.

이번에는 `const`를 통해 상수를 선언해보자.

```ts
const title = 'typescript';
```

알아서 title의 타입을 리터럴 타입인 `typescript로 추론`한다.

이런 차이가 발생하는 이유는?

`let`은 변수의 값을 언제든 바꿀 수 있고, `const`의 특성 상 변경할 수 없기 때문이다. 그래서 `let`의 경우 좀 더 포괄적인 개념의 `primitive type(string)` 으로 값을 추론하는 것이고, `const`의 경우는 변경이 불가능하기 때문에 `literal type`인 'typescript'로 추론하게 된다.

<br/>

## 타입 단언(const assertion)

`let` 변수의 경우에도 `const` 처럼 `literal type`으로 추론해줄 수 있는데, 그 때 사용하는 것이 `as const` 다.

```js
let title = 'typescript' as const;
```

title 변수에 literal type으로 'typescript'가 추론 된 것을 알 수 있다.

<br/>

## const assertion을 활용한 상수 관리하기

```js
const Colors = {
  red: '#FF0000',
  blue: '#0000FF',
  green: '#008000',
};
```

![image](https://github.com/pozafly/TIL/assets/59427983/05fa5450-5a40-488c-a112-28bd559fcea6)

Colors 변수 내부에 추론된 값을 보면 각 속성 별 리터럴 타입이 아닌 primitive type(string) 으로 추론된 것을 볼 수 있다. 이유는 `const` 변수로 `object`를 선언했지만 객체 내부의 값은 언제든 바뀔 수 있기 때문임. 이때 const assertion을 활용해 Colors 내부 값의 type을 리터럴 타입으로 변경해보자.

```ts
const Colors = {
  red: '#FF0000',
  blue: '#0000FF',
  green: '#008000',
} as const;
```

![image](https://github.com/pozafly/TIL/assets/59427983/84f34a02-53c8-4a30-afe6-ca42b852cd83)

Colors 내부의 속성들의 타입이 러터럴 타입으로 추론 된 것을 볼 수 있다. 이렇데 단언된 객체를 외부에서 import 해 사용하면 key를 자동으로 추론할 수 있게 되고, 편리하게 상수를 관리할 수 있다.

<br/>

## enum을 통한 상수 관리

```ts
export enum Colors {
  red = '#FF0000',
  blue = '#0000FF',
  green = '#008000',
}
```

enum을 통해 Colors를 선언해주었는데, const assertion과 마찬가지로 `type`이 자동으로 추론되는 것을 알 수 있다.

`enum`은 TypeScript에서 제공하는 문법으로 크게 몇 가지 특징이 있다.

- JavaScript 객체이나 내부 속성을 임의로 변경할 수 없다.
- 내부 `key`는 반드리 리터럴 타입(문자열 혹은 숫자)로만 사용 가능함.
- 속성 값 또한 마찬가지로 리터럴 타입(문자열 혹은 숫자)만 사용 가능함.

enum의 경우 ECMAScript에서 지원하는 표준 문법이 아니기 때문에 JavaScript로 transpiling 되면 원본 TS 코드와 모습이 다르다. 그러나 내부가 어떻게 돌아가는지 알 필요 없기 때문에 enum을 사용한다.
