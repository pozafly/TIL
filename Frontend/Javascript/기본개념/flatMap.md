# flatMap

> [mdn](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap)

매핑 함수를 사용해 각 엘리먼트에 대해 map 수행 후, 결과를 새로운 배열로 **평탄화** 한다. 이는 깊이 1의 [flat](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)이 뒤따른는 map메서드와 동일하지만, `flatMap`은 아주 유용하며 둘을 하나의 메소드로 병합할 때 조금 더 효율적이다.

먼저 [flat](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)을 보자.

## flat

[mdn](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flat), 모든 하위 배열 요소를 지정한 깊이까지 재귀적으로 이어붙인 **새로운 배열**을 생성한다.

```js
const newArr = arr.flat([depth]);
```

depth는 깊이를 말하는데, 기본값은 1이다. 숫자를 줄 수록 깊이를 어디까지 해서 할 수 있을지 넣어줄 수 있고, `Infinity` 를 사용하면 전부 풀어준다.

```js
const arr1 = [1, 2, [3, 4]];
const flat1 = arr1.flat();
console.log(flat1); // [ 1, 2, 3, 4 ]

const arr2 = [1, 2, [3, 4, [5, 6]]];
const flat2 = arr2.flat();
console.log(flat2); // [ 1, 2, 3, 4, [ 5, 6 ] ]

const arr3 = [1, 2, [3, 4, [5, 6]]];
const flat3 = arr3.flat(2);
console.log(flat3); // [ 1, 2, 3, 4, 5, 6 ]

const arr4 = [1, 2, [3, 4, [5, 6, [7, 8, [9, 10]]]]];
const flat4 = arr4.flat(Infinity);
console.log(flat4); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

주의 점은 `새로운 배열`을 생성해준다는 것이다. map, filter 등과 같다.

### 배열 구멍 제거

배열의 구멍도 제거해준다.

```js
const arr5 = [1, 2, , 4, 5];
const flat5 = arr5.flat();
console.log(flat5); // [ 1, 2, 4, 5 ]
```

<br />

그럼 다시 flatMap을 보면, flat 메서드에 map을 같이써서, 평탄화 하는 동시에 값을 가공할 수 있다.

```js
arr.flatMap(callback(currentValue[, index[, array]])[, thisArg])
```

- callback : map 함수에 들어갈 콜백 함수임.

```js
const arr1 = [1, 2, 3, 4];
const map1 = arr1.map((x) => [x * 2]);
console.log(map1); // [ [ 2 ], [ 4 ], [ 6 ], [ 8 ] ]

const flatmap1 = arr1.flatMap((x) => [x * 2]);
console.log(flatmap1); // [ 2, 4, 6, 8 ]

// 한 레벨만 평탄화 된다.
const flatmap2 = arr1.flatMap((x) => [[x * 2]]);
console.log(flatmap2); // [ [ 2 ], [ 4 ], [ 6 ], [ 8 ] ]
```

코드는 map 자체만을 사용해 구현할 수 있지만, 다음은 `flatMap` 의 유스케이스를 더 잘 보여주는 예시다.

```js
const arr1 = ["it's Sunny in", '', 'California'];

const map1 = arr1.map((x) => x.split(' '));
console.log(map1); // [ [ "it's", 'Sunny', 'in' ], [ '' ], [ 'California' ] ]

const flatmap1 = arr1.flatMap((x) => x.split(' '));
console.log(flatmap1); // [ "it's", 'Sunny', 'in', '', 'California' ]
```

<br />

## flatMap을 이용해 아이템 추가 및 제거

`flatMap`은 `map`을 하는 과정에서 아이템을 추가하거나 제거해 아이템 갯수를 바꿀 수 있다. `filter` 가 하는 역할의 반대 되는 개념이다. 다시 말하면, 원하는 조건으로 배열의 아이템을 원하는대로 쉽게 더 추가할 수 있다는 것을 말한다.

```js
// 배열에서 음수는 제거하고 홀수는 1과 짝수로 분리하고 싶다.
const a = [5, 4, -3, 20, 17, -33, -4, 18];
//         |\  \  x   |  | \   x   x   |
//        [4,1, 4,   20, 16, 1,       18]

a.flatMap(n => 
  (n < 0)       ? []  :  // filter 역할로 아이템을 제거함.
  (n % 2 === 0) ? [n] :  // 조건에 맞게 원하는 아이템을 배열로 집어넣음. 그러면 flat을 한번 하니 전체 요소가 반환 된다.
                  [n - 1, 1];
});
// [4, 1, 4, 20, 16, 1, 18]
```

여기에 `[...new Set(result)]` 이렇게 해주면 중복 제거까지.
