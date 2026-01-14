# Object
```ts
const x: {} = 'hello';
const y: Object = 'hi'; // {} , Object 모든 타입(null과 undefined는 제외)
const xx: object = 'hi'; // 에러. 객체가 아니다.
const yt: object = { hello: 'world' };
const z: unknown = 'hi';
```
xx만 에러가 난다. `{}`, `Object`는 모든 타입이다. 소문자로 해야지만 객체 타입이다.

하지만, `object`도 지양해야 한다. 이유는, interface, type, class 같은 것으로 프로퍼티 정의까지 해주는 것이 좋기 때문이다.
```ts
// unknown = {} | null | undefined
if (z) {
  z;
}
```
TypeScript 4.8버전 부터 unknown은, {} 또는, null 또는, undefined다.

![[assets/images/5d97c997ae5b66f8f42a7f08d6b76f17_MD5.png]]

if문 안에 있는 z에 마우스를 올렸을 때 -> `unknown`

![[assets/images/e1ba597956ff4d2e2db16d8825db063b_MD5.png]]

아래 z에 마우스를 올렸을 때 -> `{}`

따라서 객체가 아니라, 모든 타입이 된다.
