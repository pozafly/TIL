# infer

```ts
function zip(
  x: number,
  y: string,
  z: boolean
): { x: number; y: string; z: boolean } {
  return { x, y, z };
}

type Params = Parameters<typeof zip>; // 파라미터 타입만 들고온다.
type First = Params[0]; // number
```

zip 함수에서 파라미터 타입만 들고 오고 싶을 때는 어떻게 해야할까?

`Parameters` 라는 유틸리티 타입을 쓰면된다. 그러면, Params 타입은 어떻게 되나?

```ts
type Params = [x: number, y: string, z: boolean]
```

이렇게 들고 왔다. 대괄호가 붙어 있으므로, 이는 튜플 타입이다. 그러면 Parameters는 어떻게 구현되어 있을까?

```ts
type P<T extends (...args: any) => any> = T extends (...args: infer R) => any ? R : never;
```

이렇게 구현되어 있음. 분석해보자.

```ts
<T extends (...args: any) => any>
```

이 부분은 T는 무조건 함수 타입으로 제한을 건 것이다.

```ts
T extends (...args: infer R) => any ? R : never;
```

이 부분은 infer라는 키워드가 등장함. infer R은 타입스크립트가 추론한 타입을 말한다. infer는 inference(추론)을 줄여 말하는 것임.

> ※ infer는 extends 에서만 사용 가능하다.

```ts
(...args: infer R)
```

이 부분은, 매개변수를 추론한 타입이 R 이라는 말이다.

```ts
(...args: infer R) => any ? R : never;
```

코드는, 추론한 값이 있으면 R로 리턴 타입을 만들고, 추론이 안되면 쓰지마.

그러면, 매개변수 타입을 가져오는 것 말고 return 타입을 타입으로 가져가고 싶은데 만들어보자.

```ts
type R<T extends (...args: any) => any> = T extends (...args: any) => infer S ? S : never;
```

이렇게 사용할 수 있다.

<br/>

어떤 함수에서 추론되는 매개변수 타입과, 리턴 타입을 뜯어올 수 있다.