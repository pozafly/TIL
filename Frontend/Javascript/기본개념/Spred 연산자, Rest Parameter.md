# Spred 연산자, Rest Parameter

※ react를 배우면서 알게된 [Spread Operator](https://github.com/pozafly/TIL/blob/main/react/99.0%20Object%20%EA%B0%80%EB%B3%80%EC%84%B1(Spread%20Operator).md) 여기에 더 깊은 내용이 있다. 얕은 복사가 뭔지, 껍데기만 새로 만드는 개념. Spread Opertor를 다시 익히려고 들어왔으면 이걸 꼭 봐야한다.

> [출처](https://velog.io/@chlwlsdn0828/Js-Spread-%EC%97%B0%EC%82%B0%EC%9E%90-Rest-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0), [출처2](https://soopdop.github.io/2020/12/02/rest-and-spread-in-javascript/)

<br/>

📌 이름을 잘 보자. spread operator는 operator(연산자)이고, rest parameter는 parameter(매개변수)다. 즉 쓰이는 곳을 잘 봐야 한다. 연산자로 동작한다면 배열을 하나 하나의 값으로 풀어주는 거고, 매개변수로 쓰였다면 배열로 묶어준다.

<br/>

<br/>

ES6 에서부터 사용할 수 있다.

## Spred 연산자

Spread 연산자는 `…` 을 통해 사용한다.

```javascript
const arr = [1, 2, 3, 4, 5];

console.log(arr);   // [1, 2, 3, 4, 5]
console.log(...arr);   // 1, 2, 3, 4, 5
console.log(1, 2, 3, 4, 5);   // 1, 2, 3, 4 ,5
```

위와 같이 arr를 spread 연산자로 찍어보면 배열이 아닌 개별적 요소가 나온다. 간단히 생각하면 배열이 각각으로 나뉜다고 생각하면 좋음.

```javascript
const arr = [1, 2, 3, 4, 5];
const a = [...arr];

console.log(a);   // [1, 2, 3, 4, 5]
```

이 코드는 arr을 spread 연산자로 전개한 것을 다시 배열로 넣는 것임. 즉, const a = [1, 2, 3, 4, 5]; 가 되는 것.

이걸 이용해 `array.concat()` 을 대체할 수 있다.

```javascript
const aArr = [1, 2, 3];
const bArr = [4, 5, 6];

console.log(aArr.concat(bArr));   // [1, 2, 3, 4, 5, 6]
console.log([...aArr, ...bArr]);  // [1, 2, 3, 4, 5, 6]
```

이렇게 배열을 전개하는 것이 spred 연산자임.

배열 복사도 가능하다.

```javascript
const habits = [
  { id: 1, name: 'Reading', count: 0 },
  { id: 2, name: 'Running', count: 0 },
  { id: 3, name: 'Coding', count: 0 },
];

const newHabits = [...habits];

newHabits[0].count++;
console.log(habits);   // count 1 증가
console.log(newHabits);   // count 1 증가.

console.log(habits === newHabits);   // false
```

즉, count++ 을 했을 때, 기존 배열을 복사해와서(기존 배열을 `참조`만 함) 둘다 count가 1 증가한 것을 볼 수 있다. 이것은 얕은(shallow) 복사이고, 배열 안에 객체가 있는 경우에 객체 자체는 복사되지 않고 원본 값을 참조 함. 따라서 원본 배열 내의 객체를 변경하는 경우 새로운 배열 내의 객체 값도 변경되는 것임. 단, 맨 밑의 false가 뜨는 이유는 레퍼런스 값이 다르기 때문이다.

### 매우 유용한 용법들

다른 배열의 요소로 펼치기는 spread operator의 존재 이유라고 말할 수 있을 정도로 많이 활용됨. 대부분의 경우, 사용법도 헷갈리는 배열 조작 함수들을 대체할 수 있다. `push()`, `splice()`, `concat()`, `unshift()` 등을 대체 가능.

### 객체 Literal 펼치기

배열이 아니라 객체도 펼치기가 가능하다.

하나의 예로 Redux의 [Immutable Update Patterns](https://redux.js.org/recipes/structuring-reducers/immutable-update-patterns)에서 `Object.assign()`을 대체하고 있는 것을 알 수 있다. 훨씬 보기 좋은 방식으로 객체를 복사하고 복사한 객체에 필요한 property를 수정한다. Redux에서는 기존의 state object를 조작하는 매번 새로운 state를 생성하는 `immutable update` 방식을 사용하기에 이 연산자가 유용하게 사용된다.

객체의 복사

```js
const obj = { a: 1, b: 2 }
const obj1 = { ...obj } // obj객체를 펼쳐서 새로운 객체를 생성한다.
console.log(obj === obj1) // false, 같은 요소를 가졌지만 새로 생성된 객체이므로 서로 다른 객체이다.
```

객체의 연결

```js
const obj = { a: 1, b: 2 }
const obj1 = { c: 3, d: 4 }
const obj2 = { ...obj, ...obj1 } // 두 객체를 펼쳐서 새로운 객체를 생성한다.
```

객체에 새로운 property 추가, 기존 property 수정

```js
const obj = { a: 1, b: 2, c: 3 }
const obj1 = {
  ...obj,
  c: 100, // 기존의 property를 한 번더 정의해줌으로써 property를 수정할 수 있다.
  d: 200, // 새로운 property를 추가할 수 있다.
}
```

<br/>

## Rest 파라미터

Rest 파라미터도 마찬가지로 `…`을 통해 나타낸다.

spred 연산의 반대다. spread 연산은 배열을 개별적으로 전개하지만 Rest 파라미터는 개별을 배열로 묶어준다. 나머지 인자들을 배열로 합쳐서 받을 수 있게 해주는 것.

```javascript
function func(...param) {
  console.log(param);
}
func(1, 2, 3);   // [1, 2, 3]
```

내용을 찍어보면 1, 2, 3 이라는 개별 요소가 배열로 묶이는 것을 볼 수 있다. 여기서 param은 배열이기 때문에 Array 관련 메서드를 사용할 수 있음.

```javascript
function func(...param) {
  param.forEach((e) => {
    console.log(e);
  })
}
func(1, 2, 3);  // 1 2 3
```

📌 주의점

```js
function foo(a, ...rest) {
  console.log(a);  // 1
  console.log(rest); // [2, 3, 4]
}
foo(1, 2, 3, 4);
```

이렇게 파라미터를 순서대로 출력해준다.

단, rest parameter로 받은 녀석을 아래와 같이 key 로 아래 코드와 같이 일일이 풀어주게 되면 not defined 에러가 뜬다.

```js
function foo(a, ...rest) {
  console.log(a, b, c, d);  // b is not defined 
}
foo('a', 'b', 'c', 'd');  // b is not defined 
```

왜냐하면 arguments로 key를 받는게 아니라 rest로 묶어 배열로 만들어주는 것이기 때문.

### 헷갈림 유발 예제

rest parameter는 이름과 행위가 불일치하게도 반드시 함수의 인자에서만 쓰이는게 아니다. 다음 예제는 spread operator일까? rest parameter일까?

```js
const arr = ["a", "b", "c", "d"]
const [firstElement, ...rest] = arr;
console.log(firstElement) // "a"
console.log(rest) // ["b", "c", "d"]
```

답은 rest parameter임. destructuring과 함께 쓰였다. 잘 보면, [,,,]`= arr` 구문에서 구조분해 할당(풀어줌-spread)이 일어남. 후에 `…rest` 로 묶어줌. 즉 풀고 묶는 행위가 한번에 일어났다.

또한 객체에서도 똑같이 적용된다. 한 객체에서 특정 property를 뺀 새로운 객체를 만들 때 유용하다.

```js
const obj = { a: 1, b: 2, c: 3 };
const { c, ...obj2 } = obj;  // a와 b property만 가진 새로운 객체 obj2를 생성한다. 만들어진 c변수는 버린다.
console.log(c);  // 3
console.log(obj2);  // { a: 1, b: 2 }
```

<br/>

## 정리

- spread operator는 펼치고, rest parameter는 모은다.
- spread operator는 주는 쪽이고, rest parameter는 받는 쪽이다.
- spread operator는 기존의 변수를 사용하고, rest parameter는 새로운 변수를 만든다.

> *spread operator는 기존의 변수를* **펼쳐서** **주는** *쪽이고, rest parameter는 여러개의 인자를* **받고** *그것들을* **합쳐서** *새로운 배열/객체를 만든다.*
