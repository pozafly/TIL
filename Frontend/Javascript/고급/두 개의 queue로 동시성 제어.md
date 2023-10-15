# 두 개의 queue로 동시성 제어

> https://www.youtube.com/watch?v=MqjkfuqMKIg&t=2

```js
function getData() {
  fetch('https://jsonplaceholder.typicode.com/todos/1')
    .then((response) => response.json())
    .then(({ title }) => console.log(`응답: ${title}`));
}

function handleHeavyTask(bFetch) {
  if (bFetch) getData();
  else console.log('지연없이 실행');
}

(function getData() {
  console.log('안녕하세요');
  handleHeavyTask(true);
  console.log('무거운 작업이 시작됐어요');
})();
```

```
안녕하세요
무거운 작업이 시작됐어요
응답: delectus aut autem
```

만약 `handleHeavyTask` 함수에 `true` 대신 `false`를 넣는다면?

```js
(function getData() {
  console.log('안녕하세요');
  handleHeavyTask(false); // false
  console.log('무거운 작업이 시작됐어요');
})();
```

```
안녕하세요
지연없이 실행
무거운 작업이 시작됐어요
```

무거운 작업이 시작됐어요 **이전**에 지연없이 실행이 먼저 콘솔에 찍힌다. 우리가 원하는대로 코드의 실행이 보장되지 않는다.

그렇다면, `handleHeavyTask` 함수의 else 문에 작업을 비동기로 만들어보자. 무거운 작업이기 때문에 코드를 막지 않고, 최적화를 위해 비동기로 실행하는 것이다. 즉, `지연 실행` 하는 것이다.

```javascript
function handleHeavyTask(bFetch) {
  if (bFetch) getData();
  else setTimeout(() => console.log('지연없이 실행')); // setTimeout
}

(function getData() {
  console.log('안녕하세요');
  handleHeavyTask(true); // true
  console.log('무거운 작업이 시작됐어요');
})();
```

다만, 아까 `handleHeavyTask`를 먼저 true로 주고 실행해보자.

```
안녕하세요
무거운 작업이 시작됐어요
응답: delectus aut autem
```

아까와 동일한 결과. 그렇다면 `false`로 주면?

```js
function handleHeavyTask(bFetch) {
  if (bFetch) getData();
  else setTimeout(() => console.log('지연없이 실행')); // setTimeout
}

(function getData() {
  console.log('안녕하세요');
  handleHeavyTask(false); // false
  console.log('무거운 작업이 시작됐어요');
})();
```

```
안녕하세요
무거운 작업이 시작됐어요
지연없이 실행
```

이렇게 하면 원하는 결과 대로 동일한 순서를 보장하는 코드가 되었다.

그렇다면, 중간에 비동기 코드가 있다면 어떻게 될까?

```js
function getData() {
  fetch('https://jsonplaceholder.typicode.com/todos/1')
    .then((response) => response.json())
    .then(({ title }) => console.log(`응답: ${title}`));
}

function handleHeavyTask(bFetch) {
  if (bFetch) getData();
  else setTimeout(() => console.log('지연없이 실행'));
}

setTimeout(() => console.log('또 다른 비동기 코드')); // 또 다른 비동기 코드

(function getData() {
  console.log('안녕하세요');
  handleHeavyTask(false); // false
  console.log('무거운 작업이 시작됐어요');
})();
```

```
안녕하세요
무거운 작업이 시작됐어요
또 다른 비동기 코드
지연없이 실행
```

당연히, 지연없이 실행 문구 보다, 또 다른 비동기 코드가 먼저 실행되었으니 먼저 찍히게 된다.

이제, `queueMicrotask()` 함수로 대체하면 어떻게 될까?

```js
function getData() {
  fetch('https://jsonplaceholder.typicode.com/todos/1')
    .then((response) => response.json())
    .then(({ title }) => console.log(`응답: ${title}`));
}

function handleHeavyTask(bFetch) {
  if (bFetch) getData();
  else queueMicrotask(() => console.log('지연없이 실행')); // queueMicrotask
}

setTimeout(() => console.log('또 다른 비동기 코드'));

(function getData() {
  console.log('안녕하세요');
  handleHeavyTask(false);
  console.log('무거운 작업이 시작됐어요');
})();
```

```
안녕하세요
무거운 작업이 시작됐어요
지연없이 실행
또 다른 비동기 코드
```

또 다른 비동기 코드를 찍는 `setTimeout`이 먼저 호출되었음에도 불구하고, `지연없이 실행` 이 먼저 찍히게 된다.

`queueMicrotask()` 함수는, 

- 비동기를 제어하는 코드에서도 비동기의 순서를 정해주고 싶을 경우 사용한다.
- 하지만, 동기적인 코드보다는 먼저 실행되지는 않는다.

> 📌 queueMicrotask 대신 아래처럼 사용할 수도 있다. promise 콜백은 microtaskQueue에 들어가기 때문이다.
>
> ```js
> Promise.resolve().then(() => console.log('지연없이 실행'));
> ```
>
> https://pozafly.github.io/javascript/event-loop-and-async/
>
> 따라서 `.then`, `.then` `.then` 으로 연결된 녀석들이 먼저 실행되는 이유다.

