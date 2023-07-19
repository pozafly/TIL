# 효과적으로 상수 관리하기(as const)

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





















































