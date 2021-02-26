# Spread 연산자

※ react를 배우면서 알게된 [Spread Operator](https://github.com/pozafly/TIL/blob/main/react/99.0%20Object%20%EA%B0%80%EB%B3%80%EC%84%B1(Spread%20Operator).md) 여기에 더 깊은 내용이 있다. 얕은 복사가 뭔지, 껍데기만 새로 만드는 개념. Spread Opertor를 다시 익히려고 들어왔으면 이걸 꼭 봐야한다.

>출처 : https://velog.io/@chlwlsdn0828/Js-Spread-%EC%97%B0%EC%82%B0%EC%9E%90-Rest-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0

<br/>

ES6 에서부터 사용할 수 있다.

## Spred 연산자

Spread 연산자는 `...` 을 통해 사용한다.

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

<br/>

## Rest 파라미터

Rest 파라미터도 마찬가지로 `...`을 통해 나타낸다.

spred 연산의 반대다. spread 연산은 배열을 개별적으로 전개하지만 Rest 파라미터는 개별을 배열로 묶어준다.

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
func(1, 2, 3);  // 1 \n 2 \n 3
```

