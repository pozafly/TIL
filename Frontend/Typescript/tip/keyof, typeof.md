# Keyof, typeof
```ts
const obj = {
  a: '123',
  b: 'hello',
  c: 'world',
};

type Key = keyof obj; // 에러.
```
obj는 값인데, keyof는 값에는 사용할 수 없고 타입만 사용 가능하다.

![[assets/images/10c6976c21c0dbd0c5727c79710203e5_MD5.png]]

typeof는, 객체의 타입을 추론해서 뽑아준다.

![[assets/images/5ac628f5590d1bbf5d8ba40eb13c7c01_MD5.png]]

여기서 `keyof typeof` 를 사용하면, 키값 만으로 유니언을 만들어준다. `enum` 도 마찬가지다. const 어설션 붙은 enum 같은 애는
```ts
type A = typeof obj[keyof typeof obj];
```
로 사용해야 한다. value를 가져오는 것이다.
