# Array sort

> 출처 : https://hianna.tistory.com/409

<br/>

배열 정렬 메서드 sort를 알아보자.

## 1. sort() 함수

```js
arr.sort([compareFunction])
```

#### 파라미터(compareFunction)

- 정렬 순서를 정의하는 함수.
- 이 값이 `생략` 되면, 배열의 element들은 문자열로 최급되어, 유니코드 값 순서대로 정렬된다.
- 이 함수는 두 개의 배열 element를 파라미터로 받는다.
- 이 함수가 a, b 두 개의 element를 파라미터로 입력받을 경우,
  - 리턴 값이 0보다 작을 경우, a가 b보다 앞에 오도록 정렬하고,
  - 리턴 값이 0보다 클 경우, b가 a보다 앞에 오도록 정렬한다.
  - 만약 0을 리턴하면, a와 b의 순서를 변경하지 않는다.

#### 리턴값

compareFunction 규칙에 따라 정렬된 배열을 리턴함. 원본 배열인 arr가 정렬되고, 리턴하는 값 또한 원본 배열인 arr를 가리키고 있다. 즉, 새로운 배열을 리턴하는게 아니라 ⭐️ **기존 배열을 리턴함**.

<br/>

### 예제 1.

```js
const arr1 = [2, 1, 3];
const arr2 = ['b', 'a', 'o'];

const result1 = arr1.sort();
console.log(result1);  // [1, 2, 3]

const result2 = arr2.sort();
console.log(result2);  // ["a", "b", "o"]
```

### 예제 2.

```js
const arr = [2, 1, 3, 10];
arr.sort();  // [1, 10, 2, 3]
```

여기서 눈여겨 봐야할 것은 1 다음 2가 아니라 `10` 이 왔다. 유니코드 순서에 따라서 값을 정렬하기 때문. 1, 2, 3, 10 으로 정렬하고 싶다면 예제 3을 보자.

### 예제 3.

```js
const arr = [2, 1, 3, 10];

arr.sort((a, b) => {
  if (a > b) return 1;
  if (a === b) return 0;
  if (a < b) return -1;
});  // [1, 2, 3, 10]
```

배열의 숫자들은 유니코드 순서가 아닌 숫자 크기 순서대로 정렬하기 위해서 sort() 함수의 파라미터로 함수를 정의했음.

- a > b 이면 1,
- a === b 면 0,
- a < b 이면 -1 리턴으로 오름차순을 구현함.

```js
arr.sort((a, b) => {
  return a - b;
});  // [1, 2, 3, 10]
```

로 단순화 할 수 있다.

### 예제 4.

```js
const arr1 = [2, 1, 3];
const arr2 = arr1.sort();
console.log(arr1);  // [1, 2, 3]
console.log(arr2);  // [1, 2, 3]

arr1.push(4);
console.log(arr1);  // [1, 2, 3, 4]
console.log(arr2);  // [1, 2, 3, 4]
```

sort() 함수는 원본 배열을 정렬하고, 원본 배열을 가리키는 배열을 리턴한다.

<br/>

<br/>

## 2. 오름차순

```js
const arr = [2, 1, 3, 10];
arr.sort((a, b) => {
  return a - b;
});  // [1, 2, 3, 10];
```

<br/>

<br/>

## 3. 내림차순

```js
const arr = [2, 1, 3, 10];
arr.sort((a, b) => {
  return b - a;
});  // [10, 3, 2, 1]
```

<br/>

<br/>

## 4. 문자열 정렬

### 오름차순

```js
const arr = ['banana', 'b', 'boy'];
arr.sort();  // ['b', 'banana', 'boy']
```

파라미터가 입력되지 않으면 문자열의 유니코드 순서대로 정렬하기 때문에 문자열의 오름차순 정렬에는 파라미터가 입력되지 않아도 된다.

<br/>

### 내림차순

```js
const arr = ['banana', 'b', 'boy'];
arr.sort((a, b) => {
  if (a < b) return 1;
  if (a > b) return -1;
  if (a === b) return 0;
});  // ['boy', 'banana', 'b']
```

문자열을 내림차순으로 정렬하기 위해서는 sort() 의 파라미터 함수에서 두 문자열을 비교하는 내용이 들어가야 함. 참고 : [문자열 비교하기 (동등 비교, 대소 비교)](https://hianna.tistory.com/409)

<br/>

<br/>

## 5. 문자열(대소문자 구분없이) 정렬

```js
const arr = ['banana', 'b', 'Boy'];
arr.sort();  // ['Boy', 'b', 'banana']
```

유니코드는 대문자가 소문자보다 앞에 오도록 정렬이 된다. 대소문자 없이 정렬하기 위해서는 아래와 같이 하면 된다.

### (대소문자 구분없이) 오름차순

```js
const arr = ['banana', 'b', 'Boy'];

arr.sort((a, b) => {
  const upperCaseA = a.toUpperCase();
  const upperCaseB = b.toUpperCase();

  if (upperCaseA > upperCaseB) return 1;
  if (upperCaseA < upperCaseB) return -1;
  if (upperCaseA === upperCaseB) return 0;
});  // ['b', 'banana', 'Boy']
```

### (대소문자 구분없이) 내림차순

```js
const arr = ['banana', 'b', 'Boy'];

arr.sort((a, b) => {
  const upperCaseA = a.toUpperCase();
  const upperCaseB = b.toUpperCase();

  if (upperCaseA < upperCaseB) return 1;
  if (upperCaseA > upperCaseB) return -1;
  if (upperCaseA === upperCaseB) return 0;
});  // ['Boy', 'banana', 'b']
```

<br/>

<br/>

## 6. 객체 정렬

```js
const arr = [
  { name: 'banana', price: 3000 },
  { name: 'apple', price: 1000 },
  { name: 'orange', price: 500 },
];

arr.sort((a, b) => {
  return a.price - b.price;
});

// [
//   { name: 'orange', price: 500 },
//   { name: 'apple', price: 1000 },
//   { name: 'banana', price: 3000 }
// ]
```



