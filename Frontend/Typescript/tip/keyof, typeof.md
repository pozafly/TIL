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

<img width="222" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/3f1f4206-0bb6-4f16-85b0-4877a66e2321">

typeof는, 객체의 타입을 추론해서 뽑아준다.

<img width="317" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/f904b4a8-cff6-4eb4-9c43-3848ab848991">

여기서 `keyof typeof` 를 사용하면, 키값 만으로 유니언을 만들어준다. `enum` 도 마찬가지다. const 어설션 붙은 enum 같은 애는

```ts
type A = typeof obj[keyof typeof obj];
```

로 사용해야 한다. value를 가져오는 것이다.
