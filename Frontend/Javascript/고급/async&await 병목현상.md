# async&await 병목현상

> 출처

비동기 처리를 콜백의 맞물림 없이 깔끔하게 처리할 수 있도록 도와주는 문법. 동일한 역할을 하는 Promise 보다 코드가 깔끔해지고, 직관적이기 때문에 async/await를 사용한다.

하지만, async / await를 남용하면 느리지 않아도 될 작업을 느리게 만들 수 있다. 병목 현상을 개선해보자.

```js
async function a() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(), 1000);
  });
}

async function b() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(), 1000);
  });
}

async function c() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(), 1000);
  });
}
```

각각 1초 후 resolve 된다.

`console.time()`, `console.timeEnd()` 를 사용해 측정해보자.

```js
async function main() {
  console.time('function main');

  await a();
  await b();
  await c();

  console.timeEnd('function main'); // function main: 3.004s
}

main();
```

a, b, c는 Promise를 1초 후 resolve 된다. 3초가 걸렸다. 동기적으로 실행되었기 때문이다. async, await를 사용해 속도를 잃은 것.

<br/>

## Promise.all

```js
async function main() {
  console.time('function main');

  await Promise.all([a(), b(), c()]);

  console.timeEnd('function main'); // function main: 1.002s
}
```

병렬로 처리했기 때문에 시간을 단축했다.

```javascript
async function a() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(), 2000);
  });
}

async function b() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(1000), 1500); // 1000 resolve
  });
}

async function c(n) {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(), n);
  });
}
```

setTimeout의 초를 변경시키고, c는 `n`을 받도록 했다.

```js
async function main() {
  console.time('function main');

  const runA = a();
  const runB = b();

  await runA;
  const result = await runB;

  await c(result);

  console.timeEnd('function main'); // function main: 3.002s
}
```

a, b의 Promise는 동시에 실행된다. 그리고 b의 결과를 받아 c를 실행한다. 함수 b의 Promise의 결과가 1000이므로, 함수 c의 Promise는 1초후 resolve 된다.

```js
async function main() {
  console.time('function main');

  const runA = a(); // 2초 걸리는 작업
  const runB = b(); // 1.5초 걸리는 작업

  await runA; // 끝날 때 까지 2초 기다림
  const result = await runB; // 끝날 때 까지 0초 기다림(runA 보다 먼저 끝났음)

  await c(result); // 끝날 때까지 1초 기다림

  console.timeEnd('function main'); // function main: 3.002s
}
```

3초가 걸렸다. 위 코드의 문제점은 c는 b의 결과를 받지만, a가 끝날 때까지 기다린다는 것이다. 이런 경우 b와 c를 먼저 처리해줌으로써 실행시간을 단축시킬 수 있다.

```js
async function main() {
  console.time('function main');

  const runA = a(); // 2초 걸리는 작업
  const runB = b(); // 1.5초 걸리는 작업

  const result = await runB; // 끝날 때 까지 1.5초 기다림
  const runC = c(result);

  await runA; // 끝날 때 까지 0.5초 기다림(runB가 1.5초 실행되는 동안 runA도 실행중이었기 때문)
  await runC; // 끝날 때 까지 0.5초 기다림(runA가 0.5초 실행되는 동안 runC도 실행중이었기 때문)

  console.timeEnd('function main'); // function main: 2.502s
}
```
