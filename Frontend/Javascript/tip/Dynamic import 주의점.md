# Dynamic import 주의점

> https://github.com/webpack/webpack/issues/6680

## 문제상황

```ts
const path = 'src/components/some.ts';

const result = async(id) => {
  return await import(`${path}/${id}`);
}
```

cannot find module이라고 뜬다. 하지만

```ts
const result = async(id) => {
  return await import(`src/components/some.ts/${id}`);
}
```

이렇게 했을 경우 잘 불러온다.

webpack의 청크를 나누는 원리 때문인데, 빌드 시점에 정적 분석을 수행하기 때문이다. 따라서 반드시 동적 가져오기가 필요한 경우 알려진 경로로 제한해야 한다.
