# useState 비동기적 속성, 함수형 업데이트

> [출처](https://velog.io/@suyeonme/react-useState%EC%9D%98-%EB%B9%84%EB%8F%99%EA%B8%B0%EC%A0%81-%EC%86%8D%EC%84%B1-%ED%95%A8%EC%88%98%ED%98%95-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8)

## useState의 비동기적 속성과 batching

리액트에서 setState를 사용해 상태를 업데이트 할 경우, 업데이트 된 상태는 즉시 반영되지 않는다. 왜냐하면 `setState`는 비동기적으로 작동하기 때문.

즉, 리렌더링이 된 후에야 비로소 업데이트된 `state`가 반영된다. **( State 변경 - 리렌더링 - State 반영)**

리액트의 `state`를 업데이트 하는데 있어서, 비동기적으로 작동하는 속성은 여러개의 `state`를 다룰 때 퍼포먼스 측면에서 유리하다. 동기적으로 작동했다면, state1 업데이트 -> state2 업데이트 .. 이렇게 하나씩 업데이트 되니까.

여러 state를 동시에 업데이트하는 경우, 리액트는 state를 `batching` 하여 업데이트를 진행한다.

📌 `batching` : 전달된 오브젝트들을 하나로 합치는 작업. batching은 `Object Composition` 이라고도 불림.

batching을 javascript 코드로 살펴보자.

```javascript
const singleObject = Ojbect.assign({}, 
  objectFromsetState1,
  objectFromsetState2,
  objectFromsetState3
);
```

