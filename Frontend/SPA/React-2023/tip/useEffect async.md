# useEffect async

React에서 useEffect 내부 콜백 함수에 비동기 처리를 하기 위해 `async` 를 붙였더니 사진과 같은 경고가 뜬다. 아래 코드다.
```js
useEffect(async () => {
  const datae = await fetchApi();
}, []);
```
![[assets/images/40c6c4d2f47986027abc3992cf2b300d_MD5.png]]

번역해보면, useEffect 콜백은 경합 상태를 방지하기 위해 동기적으로 실행되어야 하며, 내부에 비동기 함수를 넣어야 한다. 따라서,
```js
async function fetchApi() {
  const response = await fetch('https://jsonplaceholder.typicode.com/todos/1');
  const data = await response.json();
  setDate(data);
}

useEffect(() => {
  fetchApi();
}, []);
```
이런 식으로 바꾸어 주었다. 굳이 React가 이런 식으로 처리하는 이유가 있을까? useEffect 내부에서는 왜 비동기 관련 코드를 넣지 못하게 의도적으로 막을까?

답은, useEffect의 인자로 들어가는 함수는 어떤 값로 return 하면 안되지만, `async`를 붙이면서 Promise를 반환하기 때문이다.

> [이곳](https://velog.io/@he0_077/useEffect-%ED%9B%85%EC%97%90%EC%84%9C-async-await-%ED%95%A8%EC%88%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0) 참고

useEffect가 반환할 수 있는 것은 오직 클린업 함수 뿐이다.
