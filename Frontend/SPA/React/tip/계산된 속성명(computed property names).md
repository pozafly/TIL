# 계산된 속성명(computed property names)

ES6에 나온 문법. 객체의 속성명을 동적으로 결정하기 위해 나온 문법.

```js
function makeObject1(key, value) {
  const obj = {};
  obj[key] = value;
  return obj;
}
function makeObject2(key, value) {
  return { [key]: value };
}
```

계산된 속성명을 사용하면 같은 함수를 2 처럼 간결하게 작성할 수 있다.

계산된 속성명은 다음과 같이 컴포넌트의 상탯값을 변경할 때 유용하게 쓸 수 있다.

```jsx
class MyComponent extends React.Component {
  state = {
    count1: 0,
    count2: 0,
    count3: 0,
  };
  // ...
  onClick = index => {
    const key = `count${index}`;
    const value = this.state[key];
    this.setState({ [key]: value + 1 });   // 📌
  };
}
```

📌 setState 호출 시, 계산된 속성명을 사용할 수 있다. 만약 계산된 속성명을 사용하지 않았다면 앞의 코드는 좀 더 복잡했을 것임.
