# Async & await

> 출처: https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#asyncawait

비동기 코드를 작성하는 새로운 방법. Javascript 개발자들이 훌륭한 비동기 처리 방안이 Promise로 만족하지 못하고 더 훌륭한 방법을 고안한 것임. async / await는 Promise 기반이다. 절차적 언어에서 작성하는 코드와 같이 사용법도 간단하고 이해도 쉽다.

- function 키워드 앞에 `async`를 붙여주면 되고
- function 내부의 promise를 반환하는 비동기 처리 함수 앞에 `await` 를 붙여주기만 하면 됨.

async / await 의 가장 큰 장점은 Promise 보다 비동기 코드의 겉모습을 더 깔끔하게 한다는 것. es8의 공식 스팩이면 node8LTS 에서 지원됨.

- `Promise` 로 구현

```javascript
function makeRequest() {
  return getData()
    .then(data => {
      if (data && data.needMoreRequest) {
        return makeMoreRequest(data)
          .then(moreData => {
            console.log(moreData);
            return moreData;
          }).catch ((error) => {
            console.log('Error while makeMoreRequest', error);
          });
      } else {
        console.log(data);
        return data;
      }
    }).catch ((error) => {
      console.log('Error while getData', error);
    });
}
```

- `async / await` 구현

```javascript
async function makeRequest() {
  try {
    const data = await getData();
    if (data && data.needMoreRequest) {
      const moreData = await makeMoreRequest(data);
      console.log(moreData);
      return moreData;
    } else {
      console.log(data);
      return data;
    }
  } catch (error) {
    console.log('Error while getData', error);
  }
}
```

<br/>

더 알아보자.

> 출처: [자바스크립트의 Async/Await 가 Promises를 사라지게 만들 수 있는 6가지 이유](https://medium.com/@constell99/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-async-await-%EA%B0%80-promises%EB%A5%BC-%EC%82%AC%EB%9D%BC%EC%A7%80%EA%B2%8C-%EB%A7%8C%EB%93%A4-%EC%88%98-%EC%9E%88%EB%8A%94-6%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0-c5fe0add656c)

- async / await 는 실제로는 최상위에 위치한 promise 에 대해 사용하게 된다. plain callback 이나 node callback과 함께 사용할 수 없다.
- async / await 는 promise 처럼 none-bloking 이다.
- 하지만 비동기 코드의 겉모습과 동작을 좀 더 동기 코드와 유사하게 만들어 준다. 이게 async / await의 가장 큰 장점.

다음은 promise 구현

```javascript
// promise
const makeRequest = () => {
  getJSON()
    .then(data => {
      console.log(data);
      return 'done';
    });
}

makeRequest();
```

async / await 구현

```javascript
// async / await
const makeRequest = async () => {
  console.log(await getJSON());
  return "done";
}

makeRequest();
```

이 둘 사이의 차이점이 있다.

1. 함수의 앞에 `async` 라는 단어가 오게 됨. `await` 는 오직 `async` 로 정의된 함수의 내부에서만 사용될 수 있다. 모든 `async` 함수는 암묵적으로 promise 를 반환하고, promise가 함수로부터 반환할 값 (예제에서는 `'done'` 이라는 문자열) 을 resolve 함.
2. 위와 같은 점 때문에 `await` 를 우리 코드 탑 레벨에서는 사용할 수 없다. `async` 함수 안에 위치한 경우에만 사용 가능하다.

   ```javascript
   // 바로 위의 코드와 이어서 생각해보자.
   // 탑 레벨에서는 동작하지 않음.
   // await makeRequest()
   
   // 여기서는 동작함.
   makeRequest().then((result) => {
     // do something
   });
   ```

3. `await getJSON()` 은 `console.log` 의 호출이 `getJSON()` promise가 resolve된 후에 일어나고, 그 후에 값을 출력할 것이라는 것을 의미함.

> 즉, 스코프 문제라는 것인데, async 스코프 내, await를 사용해야 하고, 그 상위 스코프에서는 안먹는다는 말인듯.

<br/>

## 비동기 함수를 병렬로 실행하기

async await 함수에서 여러 비동기 함수를 병렬로 처리하는 방법을 알아보자. 다음과 같이 여러 비동기 함수에 각각 await 키워드를 사용해서 호출하면 순차적으로 실행된다.

```js
// 순차적으로 실행되는 비동기 코드
async function getData() {
  const data1 = await asyncFunc1();
  const data2 = await asyncFunc2();
  // ...
}
```

위 코드에서 두 함수 사이에 의존성이 없다면 동시에 실행하는게 더 좋다. 프로미스는 생성과 동시에 비동기 코드가 실행된다. 따라서 두 개의 프로미스를 먼저 생성하고 await 키워드를 나중에 사용하면 병렬로 실행하는 코드가 된다.

```js
// await 키워드를 나중에 사용해서 병렬로 실행되는 비동기 코드
async function getData() {
  const p1 = asyncFunc1();  // 👻 
  const p2 = asyncFunc2();  // 👻
  const data1 = await p1;  // 📌
  const data2 = await p2;  // 📌
}
```

- 👻: 두개의 프로미스가 생성되고 각자의 비동기 코드가 실행된다.
- 📌: 두 프로미스가 생성된 후 기다리기 때문에 두 개의 비동기 함수가 병렬로 처리 된다.

Promise.all을 사용하면 다음과 같이 더 간단해진다.

```js
async function getData() {
  const [data1, data2] = await Promise.all([asyncFunc1(), asyncFunc2()]);
  // ...
}
```

<br/>

> `정리`
>
> async / await는 promise 패턴에서 엄청 깔끔하게 코드를 만들어준다. 근데 이걸 남용하면 가끔 promise 객체를 리턴하는데 붙이는게 아니라 일반 메서드에 붙일 경우가 생기는데 이를 주의하기만 하면 절차지향적 언어로 보이게끔 만들어주어 참 좋네.
