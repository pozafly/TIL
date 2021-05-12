# Array 기본 메서드

> 출처 : https://www.zerocho.com/category/JavaScript/post/57387a9f715202c8679b3af0

- length
- join
- concat
- reverse

## 배열.length

배열의 길이 return

<br/>

## 배열.join(구분자)

배열의 항목들을 구분자를 기준으로 합친 새 문자열을 반환함. 구분자를 입력하지 않으면 자동으로 쉼표임.

```javascript
var array = [1, 2, 3];
array.join();   // "1,2,3"
array.join(':');   // "1:2:3"
```

<br/>

## 배열.concat(합칠 것)

배열을 합친 새 배열 반환.

```javascript
var array = [1, 2, 3];
array = array.concat(4, 5);   // [1, 2, 3, 4, 5]
array = array.concat([6, 7]);  // [1, 2, 3, 4, 5, 6, 7]
```

<br/>

## 배열.reverse()

원래의 배열을 뒤집음.

```javascript
var array = [1, 2, 3];
array.reverse();   // [3, 2, 1]
```

<br/>

## 추가 삭제(push, pop, unshift, shift)

전에 작성한 대로 push, pop은 뒤에서부터, unshift, shift는 배열의 앞에서 추가하거나 뺌. 따라서 구조 상 앞에서 추가하거나 빼면 확실히 느려진다는 사실을 잊지말자.

단, push는 push한 후 배열의 길이를 return, pop은 마지막 요소를 제거 후 그 요소를 반환. 마찬가지로 unshift는 변한 배열의 길이를, shift는 shift된 항목을 반환.

```javascript
var array = [1, 2, 3];
array.push(4);    // 4
array;            // [1, 2, 3, 4]
array.pop();      // [4]
```

```javascript
var array = [1, 2, 3];
array.unshift(0);  // 4
array;             // [0, 1, 2, 3]
array.shift();     // [0]
```

<br/>

## 배열.splice(시작점, 지울갯수, 넣을 것)

pop, shift는 배열의 처음 혹은 끝에서만 뺄 수 있지만 이놈은 중간에서 빼고 새로 추가할 수 있다.

```javascript
var array = [1, 2, 3, 4];
array.splice(2, 1, 5);  // 3
array;                  // [1, 2, 5, 4];
```

<br/>

## 배열.sort(function(이전 값, 다음 값) {조건})

배열을 특정 조건에 따라 정렬함.

```javascript
var array = [5, 2, 3, 4, 1];
array.sort(function(x, y) {
  return x - y;
});    // [1, 2, 3, 4, 5]
```

<br/>

## 배열.indexOf(찾을 것), 배열.lastIndexOf(찾을 것)

문자열 처럼 배열에서 찾는다. 여러 개가 있더라도 처음으로 찾은 위치만 반환함. lastIndexOf은 뒤에서부터 찾는다.

<br/>

## 배열.every(function(항목) {조건}), 배열.some(function(항목) {조건})

배열의 모든 항목 또는 일부 항목이 true이면 true 반환함. 즉, `every`는 모든 항목이 조건을 만족하면 true, `some`은 하나의 항목이라도 조건을 만족하면 true를 반환함.

```javascript
var array = [1, 3, 5, 7, 9];
array.every(function(i) {
  return i % 2 === 1;  // 홀수 찾기
});   // true

array.every(function(i) {
  return i < 9;
});   // false

array.some(function(i) {
  return i === 9;
});   // true
```

첫 예는 array의 값이 모두 홀수 이기 때문에 true. 하지만 그 다음 예는 조건은 모두 만족하지는 않기 때문에 false. some은 배열 요소 중 9인게 하나 이상 있기 때문에 true.

<br/>

## Array.isArray(값)

Array 객체 자체의 static 메서드임. 배열인지 아닌지 확인해주는 역할.

```javascript
Array.isArray('array?');    // false
Array.isArray(['array?']);  // true
```

