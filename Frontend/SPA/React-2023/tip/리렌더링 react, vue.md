# 리렌더링 react, vue

## Vue

### Vue2(Object.defineProperty)

[Object.defineProperty()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 정적 메서드는 객체에 새로운 속성을 직접 정의하거나 이미 존재하는 속성을 수정한 후, 해당 객체를 반환한다.

```js
const obj1 = {};

Object.defineProperty(obj1, 'property1', {
  value: 42,
  writable: false,
});

obj1.property1 = 77;
console.log(obj1.property1); // 42
```

`Object.defineProperty(obj, prop, descriptor)`

- 첫번째 매개변수(obj): 속성을 정의할 객체.
- 두번째 매개변수(prop): 새로 정의하거나 수정할 속성의 이름(key)
- 세번째 매개변수(descriptor): 새로 정의하거나 수정하려는 속성을 기술하는 객체.

객체의 속성을 세밀하게 추가하거나 수정할 수 있다. 할당을 통해 속성을 추가하는 일반적인 방법으로 추가한 속성은, 객체의 속성을 열거할 때 노출되며 원하는대로 값을 변경하고 delete 연산자로 삭제할 수 있다.

하지만, defineProperty로 정의한 속성은 기본 동작을 상세하게 조절할 수 있다.

데이터 서술자(data dscriptors), 접근자 서술자(access dscriptors) 두 가지 형식을 취할 수 있다.

- 데이터 서술자: 값을 가지는 속성을 기술
  - value: 값
  - writable: 할당 연산자
- 접근자 서술자: 접근자(getter)-설정자(setter) 함수를 한 쌍으로 가지는 속성을 기술할 때 사용한다.
  - get: 속성의 접근자.
  - set: 속성의 설정자.

---

Vue2에서는 Object.defineProperty를 이용해서 getter, setter를 만든다.

getter, setter를 이용해 vue의 상태값(data)에 값을 변경 시키면 setter를 통해 vue가 UI를 새로 그려주는 방식. 그러면 setter를 통해 어떻게 Vue 인스턴스가 이것을 알 수 있는가?

---

### Vue3(proxy)

<br/>

## React

setState를 통해 상태 값 변경을 알려준다.

그러면 어떤 장점이 있는지 고민해보는 시간 가져라.
