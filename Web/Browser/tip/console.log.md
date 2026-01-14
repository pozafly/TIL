# console.log

> 출처: https://www.youtube.com/watch?v=KxsVV5jbJe4

<br/>

디버깅에 유용하다.

## Log level

아래의 4가지로 표현될 수 있다.

### console.log()

```js
const dog = { type: '🐶', name: '츄츄', owner: { name: 'hst' } };
console.log('logging', dog);
// logging {type: "🐶", name: "츄츄", owner: {…}}
```

이렇게 표현된다. 단순 디버깅 용.

따라서 배포 시 삭제하고 혹은 출력되지 않도록 하고 배포하는 것이 좋음. 출력하는 것이기 때문에 성능에 영향을 줄 수 있기 때문이다.

<br/>

### console.info()

정보를 나타내기 위함. log와 마찬가지로 배포 시 나타나지 않도록 하자.

```js
console.info(dog);
```

<br/>

### console.warn()

경보.

```
console.warn(dog);
```

![[assets/images/8b2cd830c0a01b68f709d77c0e31c3f1_MD5.png]]

이런식으로 노란색으로 표시된다.

<br/>

### console.error()

에러! 예상하지 못한 에러. 시스템 에러.

![[assets/images/6d4b85bf17a7b67d5c74f11546811aa3_MD5.png]]

log, info는 배포 시 표현되지 않도록 하고, warn, error만 어떤 특별한 동작을 할 수 있게 한다. 따라서 level에 따라서 잘 작성해주는 것이 좋다.

<br/>

<br/>

## 유용한 다른 콘솔

### console.assert()

특정한 상황일 때 log를 남기게 할 수 있다.

```js
console.assert(2 === 3, 'not same!');
console.assert(2 === 2, 'same!');
// Assertion failed: not same!
```

즉, `2 === 2` 는 출력이 되지 않고, `2 === 3` 은 다르니까 not same이 빨간색으로 표시되어 나오게 된다.

<br/>

### console.table()

```js
console.table(dog);
```

![[assets/images/2069717b6af9aa9b50d01e27c59d38a6_MD5.png]]

이런 식으로 가독성 좋게 표시해준다.

<br/>

### console.dir()

얼마나 깊이 있게 중첩된 데이터를 표현할 것인지 지정 가능.

```js
console.dir(dog, { colors: false, depth: 1 });
```

<br/>

### console.time()

코드를 실행하는데 얼마나 시간이 걸리는지 측정할 수 있다.

```js
console.time('for loop');
for (let i = 0; i < 10; i++) {
  i++;
}
console.timeEnd('for loop');
// for loop: 0.002197265625 ms
```

끝낼 때는 `timeEnd` 를 해주면 된다.

<br/>

### console.count()

몇번 불렸는지 count를 세어줌. 개인적으로는 이게 가장 유용할 듯.

```js
let count = 0;
function a() {
  count++;
}
a();
a();
a();
console.log(`a 함수는 ${count}번 실행됨`);
// a 함수는 3번 실행됨
```

이렇게 일일이 셀 필요 없고,

```js
function a() {
  console.count('a function');
}
a();
a();
a();
// a function: 1
// a function: 2
// a function: 3
```

이렇게 손쉽게 출력된다. 만약 이 카운트를 초기화 하고 싶다면,

```js
function a() {
  console.count('a function');
}
a();
a();
console.countReset('a function');
a();
// a function: 1
// a function: 2
// a function: 1
```

이렇게 `countReset` 을 사용해서 해줄 수 있다.

<br/>

### console.trace()

추적해준다.

```js
function f1() {
  f2();
}
function f2() {
  f3();
}
function f3() {
  console.trace();
  console.log('hi! :)');
}
f1();
```

![[assets/images/3719f4682aea9ef92e1b8082160cdc19_MD5.png]]

이런 식으로 어떤 함수에서 어떻게 부르는지, 또 몇번째 줄에서 코드가 적혀있는지 추적해준다. **비동기** 코드를 추적하기 쉬워진다. 매우 유용하겠지?
