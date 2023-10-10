# setTimeout으로 코드 성능 최적화하기

> [출처](https://www.youtube.com/watch?v=BhlX-NcIrDU)

면접 질문 중, setTimeout을 타이머 맞추는 기능 말고 다른 기능도 사용하실 수 있으세요? 라는 질문이다. 이 질문에 대한 답은, Thread 연산량이 많은 동기 작업이 있을 경우 Main Thread를 차단하지 않게 비동기로 작업을 나누어 처리하므로써 성능 최적화를 할 수 있다고 답할 수 있다.

setTimout은 기본적으로 타이머를 맞추는 기능이다.

```js
setTimeout(() => {
  /* Do something */
}, 3000)
```

코드 최적화를 위해 사용할 수도 있다.

```js
function startCPUHeavyTaskWithoutOptimization() {
  let result = 0;
  for (let i = 0; i < 30000000; i += 1) {
    result += 1;
  }
  console.log('[Not Optimized] done!!!', result);
  return result;
}
```

30억건의 for문을 돈다. CPU 자원을 많이 먹는 동기적인 작업이다. 브라우저가 먹통이 된다.

<img width="1179" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/36b09f91-511b-45da-b954-12059e825b2b">

Main Thread는 싱글 쓰레드다. Main Thread가 할일을 모아둔 곳이 Event Queue라는 곳이다. Task를 하나씩 순차적으로 처리해나간다. 동기적인 작업이기 때문에 버튼 클릭 등 먹통이 되는 것이다.

해결방법은 30억건을 나눠서 사용하는 것이다.

```js
function startCPUHeavyTaskWithOptimizationProtoType() {
  let result = 0;
  for (let i = 0; i < 3; i += 1) {
    setTimeout(() => {
      for (let j = 0; j < 10000000; j += 1) {
        result += 1;
      }
      console.log('[Optimized Basic] loop', result);
    });
  }
  console.log('[Optimized Basic] done!!!', result);
  return result;
}
```

10억건씩 3번 돌리는 것이다.

하지만, 문제점은 하단 console의 `[Optimized Basic] done!!!` 이부분이 가장 먼저 찍히는 문제점이 발생한다. 이유는, setTimeout은 비동기이기 때문이다.

여기서 유의할 점은, 첫번째 `for` 문, 3번 돌리는 저 for문은 단지 setTimeout을 실행시키고 넘어가버리기 때문에 동기적으로 done이 찍히는 게 맞다. 이미 실행이 끝났고, setTimeout만 Task queue에 남아 비동기로 실행되는 것임.

```js
async function startCPUHeavyTaskWithOptimizationPromise() {
  let result = 0;
  for (let i = 0; i < 3; i += 1) {
    await new Promise((res) => {
      setTimeout(() => {
        for (let j = 0; j < 10000000; j += 1) {
          result += 1;
        }
        console.log('[Optimized Basic] loop', result);
        res();
      });
    });
  }
  console.log('[Optimized Basic] done!!!', result);
  return result;
}
```

`await new Promise`로 감쌌다. 10억번을 **기다렸다가** 돌고, 또 10억번 돌때까지 기다렸다가 돌고. 가장 마지막에 done이 찍히게 된다.

<img width="1389" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/64f2eaa8-6ec8-4556-95d8-190dd20817d4">

이미지를 보자. setTimeout 안에 있는 for 문이 있을 때는 버튼 핸들러가 동작을 하지 않지만 도중에 버튼 핸들러를 처리할 수 있는 틈이 생긴다. 그렇기 때문에 30억 모두 처리할 때까지 Main Thread가 기다리는 것이 아니라 나누어 동작할 수 있게 해주고, Queue 중간에 틈에 이벤트가 존재한다면 실행할 수 있도록 해주는 것이다.