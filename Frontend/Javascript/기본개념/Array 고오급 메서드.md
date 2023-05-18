# Array 고오급 메서드

> 출처 : https://velog.io/@nayeon/Array%EC%9D%98-map-filter-reduce-forEach-%EB%A9%94%EC%86%8C%EB%93%9C, https://daesuni.github.io/Loop-performance/, https://www.zerocho.com/category/JavaScript/post/5acafb05f24445001b8d796d

종류

- map
- filter
- reduce
- forEach

## 간단 비교

- forEach : 한개씩 돌면서 무언가 하기. return value 없음.
- filter : 조건에 맞는 것만 새로운 배열로. return value는 새 배열
- map : 한 개씩 돌면서 연산한 결과를 새로운 배열로. return value는 새 배열
- reduce : 한 개씩 돌면서 이전 연산한 결과를 조합하여 사용. return value는 reduce 함수 안에서 설정한대로.

## 공통점

위의 메서드중 return되는 녀석들은 `사본`을 반환하며 **원래의 배열을 바뀌지 않는다**.

<br/>

## map

배열 내의 모든 요소 각각에 대해 주어진 함수를 호출한 결과를 가진 `새로운 배열`을 만들어낸다. 반복문을 돌면서 배열 안의 요소들을 1:1로 짝지어 주는 것. 매핑한다고 생각해라.

```javascript
let newArr = arr.map(function callback(현재값 [, index [, array]]) {
  // return newArr를 위한 요소
});
```

- `callback` 은 여기서 arr의 모든 요소를 호출하는 함수. 그 결과가 newArr 에 더해짐.
- 현재 값으로는 array안에서 처리될 현재 요소를 넣어준다. ex) element 같은..
- return 값으로는 newArr에 function을 callback한 결과가 각각의 요소에 적용되어 담기게 될 값을 적용함.

```javascript
let numbers = [1, 2, 3];
let newArr = numbers.map(function(num) {
  return Math.pow(num, 2);  // 2 제곱
});
console.log(numbers);  // 1, 2, 3
console.log(newArr);    // 1, 4, 9
```

<br/>

## filter

주어진 function 에 속한 조건을 통과한 요소들을 `새로운 배열` 로 반환.

```javascript
let newArray = arr.filter(callback(crr [, index [, array]]) {
  return // 새로운 배열에 포함되기 위해 통과할 조건
});
```

- 가장 큰 특징은 boolean 형태의 return 값을 가짐.
- return 값이 true일 경우 그 요소를 반환하고, false일 경우 반환하지 않는다.
- 깔끔하게 원하는 요소들만 필터링 할 수 있는 메서드임.

```javascript
function isBigEnough(value) {
  return value >= 10
}

let filtered = [12, 5, 8, 130, 44].filter(isBigEnough);
console.log(filtered);    // [12, 130, 44]
```

<br/>

## forEach

주어진 함수를 한번씩 각각의 array 요소들에게 실행함. (for loop를 생각하면 좋음.)

```javascript
arr.forEach(callback(현재값 [, index [, array]]) {
  // 배열의 요소에 가할 행위
});
```

순회하며 돈다.

- for 문 보다 가독성이 좋다.
- 복잡한 객체를 처리하는데 있어 유리함.
- return 값을 받지 못한다. return 한다면 undefined를 받게 됨.
- for 문과 다르게 중간에 끊을 방법이 없다. ( return으로 skip은 가능함. )

```javascript
let array1 = ['a', 'b', 'c'];
array1.forEach(element => console.log(element));

// a
// b
// c
```

<br/>

## reduce

배열의 각 요소에 대해 주어진 reducer 함수를 실행하고, 하나의 결과값을 반환한다. **배열 축소** 의 원리로 작용함. 즉 여러개의 값이 담긴 배열이 줄어 최종적으로 하나의 값으로 만드는 과정.

```javascript
arr.reduce(callback(누적값, 현재값 [, currentIndex [, array]]) {
  return // 누적 결과의 값
}, [초깃값])
```

- 누적값은 콜백의 반환값을 누적한다.

```javascript
const oneTwoThree = [1, 2, 3];

let result = oneTwoThree.reduce((acc, cur, idx) => {
  console.log(acc, cur, idx);
  return acc + cur;
}, 0);

// 0 1 0
// 1 2 1
// 3 3 2

console.log(result);   // 6
```

acc(누적값)dl 초깃값인 0부터 시작해 return 하는 대로 누적되는 것을 볼 수 있다. 초깃값을 적어주지 않으면 0번째 인덱스 값이 됨.

※ 만약 초깃값인 0 을 적어주지 않으면

```javascript
// 1 2 1
// 3 3 2
// 6
```

이렇게 나온다. 즉, 초깃값이 없으면 -1 번 돈다. 배열이 3개라면 2번 돈다.

reduce는 덧셈 곱셈만을 위한 것이 아님. 초깃값을 활용하여 map과 filter와 같은 함수형 메서드를 모두 reduce로 구현할 수 있다. 위의 예제로 map 예제를 reduce로 만들어보면,

- map 버전

```javascript
const oneTwoThree = [1, 2, 3];

let result = oneTwoThree.map((v) => {
  if (v % 2) {
    return '홀수';
  }
  return '짝수';
});

console.log(result);   // (3) ["홀수", "짝수", "홀수"]
```

- reduce 버전

```javascript
const oneTwoThree = [1, 2, 3];

let result = oneTwoThree.reduce((acc, cur) => {
  acc.push(cur % 2 ? '홀수' : '짝수');
  return acc;
}, []);

console.log(result);   // (3) ["홀수", "짝수", "홀수"]
```

초깃 값을 배열로 만들고, 배열에 값들을 push하면 map과 같아짐. 이를 응용하여 조건부로 push 하면 filter와 같아진다. 다음은 홀수만 필터링 하는 코드.

```javascript
const oneTwoThree = [1, 2, 3];

let result = oneTwoThree.reduce((acc, cur) => {
  if(cur % 2) acc.push(cur);
  return acc;
}, []);

console.log(result);   // (2) [1, 3]
```

이와 같이 sort, every, some, find, findIndex, includes도 모두 reduce로 구현 가능함. 

reduce는 비동기 프로그래밍을 할 때도 유용함.

```javascript
const promiseFactory = (time) => {
  return new Promise((resolve, reject) => {
    console.log(time);
    setTimeout(resolve, time);
  });
};

[1000, 2000, 3000, 4000].reduce((acc, cur) => {
  return acc.then(() => promiseFactory(cur));
}, Promise.resolve());   // 초깃값

// 바로 1000
// 1초 후 2000
// 2초 후 3000
// 3초 후 4000
```

1. 초깃값은 Promise.resolve() 로 한 후에, 
2. return된 프로미스에  then을 붙여 다음 누적 값으로 넘기면 된다.
3. 프로미스가 순차적으로 실행을 보장됨.

반복되는 모든 것에 reduce를 쓸 수 있다. 