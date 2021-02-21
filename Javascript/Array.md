# Array

> 출처 : https://youtu.be/1Lbr29tzAA8

- 자료구조란?
  - 비슷한 종류의 data를 바구니에 담는 것.

- Object와 자료구조의 차이점?
  - 토끼와 당근 각각은 Object 임. Object는 서로 연관된 특징과 행동들을 한데 묶는 것.
  - 이런 비슷한 Object 들을 묶는 것을 자료구조.

<br/>

## 1. Declaraion

```javascript
const arr1 = new Array();
const arr2 = [1, 2];
```

<br/>

## 2. Index position

```javascript
const fruits = ['사과', '바나나']
console.log(fruits);  // (2) ["사과", "바나나"]
console.log(fruits.length);  // 2
console.log(fruits[0]);  // 사과
console.log(fruits[1]);  // 바나나
console.log(fruits[2]);  // undefined
console.log(fruits[fruits.length - 1]);  // 바나나
```

<br/>

## 3. Looping over an array

```javascript
for(let fruit of fruits) {
  console.log(fruit);   // 사과 /n 바나나
}
```

```javascript
fruits.forEach((fruit, index, array) => {
  console.log(fruit);  // 사과 /n 바나나
  console.log(index);  // 0 /n 1
  console.log(array);  // (2) ["사과", "바나나"] 두번 출력됨.
});
```

<br/>

## 4. Addtion, deletion, copy

- push : 배열의 뒤에서 item 추가

```javascript
fruits.push('딸기', '복숭아');
console.log(fruits);  // (4) ["사과", "바나나", "딸기", "복숭아"]
```

- pop : 배열의 뒤에서 item 제거

```javascript
fruits.pop();
console.log(fruits);  // (3) ["사과", "바나나", "딸기"]
```

- unshift : 배열의 앞에서 item 추가

```javascript
fruits.pop('레몬');
console.log(fruits);  // (4) ["레몬", "사과", "바나나", "딸기"]
```

- shift : 배열의 앞에서 item 제거

```javascript
fruits.shift();
console.log(fruits);  // (3) ["사과", "바나나", "딸기"]
```

- ⭐️ 중요 노트!!!
  - shift와 unshift는 push, pop 보다 정말 정말 느리다!!!
  - 뒤에서 부터 빼고 지우는 것은 기존의 item들이 자리를 움직이지 않아도 가능.
  - 하지만 앞에서 작업한다면 기존 데이터들을 자리를 옮겨가면서 수정해야 하므로.
  - 참고로, 중간에 넣고 빼는 작업은 index 덕에 빠름.

- splice : 배열의 특정 값을 지움
  - splice(어디서부터지움?, [몇개지움?], [뭐넣을래?], [뭐넣을래?]...)

```javascript
fruits.splice(1, 1);
console.log(fruits);  // (2) ["사과", "딸기"]
fruits.splice(0, 1, '풋사과', '바나나');
console.log(fruits);  // (3) ["풋사과", "바나나", "딸기"]
```

여기서 보면, splice 후 추가될 자기는 방금 지워진, (0,1)의 사과의 자리에 풋사과와 바나나가 들어감.

- concat : 배열 합침

```javascript
const fruits2 = ['배', '코코넛'];
const newFruits = fruits.concat(fruits);
console.log(newFruits);   // (5) ["풋사과", "바나나", "딸기", "배", "코코넛"]
```

<br/>

## 5. Searching

- indexOf : 몇번째?
- includes : 포함되었니?
- lastIndexOf('값') : 제일 마지막에 포함된 '값'은 몇번째 인덱스니?

```javascript
console.log(fruits.indexOf('풋사과'));   // 0
console.log(fruits.indexOf('딸기'));    // 2
console.log(fruits.indexOf('포도'));    // -1

console.log(fruits.includes('배'));    // true
console.log(fruits.includes('포도'));   // false

fruits.push('풋사과');
console.log(fruits.lastIndexOf('풋사과'));  // 5
```



<br/>

>`정리`
>
>좋네. array. 뭐라고 정리해야할지.. 익숙하여서...

