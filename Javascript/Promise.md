# Promise

<br/>

## Promise 란?

> 출처 : https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#promise

자바스크립트에서는 대부분의 작업들이 비동기로 이루어진다. 콜백 함수로 처리되면 되는 문제였지만 요즘엔 프론트의 규모가 커지면서 코드의 복잡도가 높아지는 상황이 발생. 콜백이 중첩되는 경우가 발생. 이를 해결할 방안으로 등장한 것이 Promise 패턴이다. Promise 패턴을 사용하면 비동기 작업들을 순차적으로 진행하거나, 병렬로 진행하는 등의 컨트롤이 보다 수월해진다. 또한 예외처리에 대한 구조가 존재하기 때문에 오류 처리 등에 대해 보다 가시적으로 관리할 수 있음. 이 Promise 패턴은 ECMAScript6 스펙에 정식으로 포함되었다.

<br/>

> 출처 : https://programmingsummaries.tistory.com/325

비동기로 이루어진 작업을 요청하면서 콜백 함수를 등록하면, 작업이 수행되고 나서 결과를 나중에 콜백 함수를 통해 알려줌. 복잡도가 높아지는 상황에서 어려워지는 케이스는 `콜백 중첩` 되는 경우임. 콜백 지옥이라고 함.

```javascript
async(1. function() {
  async(2. function() {
    async(3. function() {
      async(4. function() {
        console.log('작업완료??');
      });
    });
  });
});
```

Promise 패턴을 사용하면 비동기 작업들을 순차적으로 진행하거나, 병렬로 진행하는 등 컨트롤이 보다 수월해지고 코드의 가독성이 좋아짐. 내부적으로 예외 처리에 대한 구조가 탄탄하기 때문에 오류가 발생했을 때 오류 처리 등에 대해 보다 가시적으로 관리 가능.

<br/>

## 기초

```javascript
var _promise = function(param) {
  return new Promise(function(resolve, reject) {
    
    // 비동기를 표현하기 위해 setTimeout 함수 사용
    window.setTimeout(function() {
      
      // 파라미터가 참이면
      if(param) {
        // 해결됨
        resolve('해결완료');
      } else {
        // 거짓이면,실패
        reject(Error('실패'));
      }
    }, 3000);
  });
};

_promise(true)
  .then(function(text) {
    // 성공 시
    console.log(text);
  }, function(error) {
    // 실패 시 
    console.log(error)
  });
```

실행하면 결과는 '해결 완료' 가 뜬다.

<br/>

## 선언부

Promise는 말 그대로 '약속' 이다. 지금은 없으니까 나중에 줄게. 더 정확히는 '지금은 없는데 이상 없으면 나중에 주고 없으면 알려줄게'.

따라서 Promise는 다음 중 하나의 상태(state)가 됨.

- pending
  - 아직 약속을 수행 중인 상태(fulfilled 혹은 reject 가 되기 전).
- fulfilled
  - 약속(promise)가 지켜진 상태
- reject
  - 약속(promise)가 어떤 이유에서 못 지켜진 상태
- settled
  - 약속이 지켜졌든 안지켜졌든 일단 결론이 난 상태.

Promise 선언부를 보면, 나중에 Promise 객체를 생성하기 위해 Promise 객체를 리턴하도록 함수로 감싸고 있다. Promise 객체만 보면 파라미터로 익명함수를 담고 있고, 익명 함수는 resolve, reject를 파라미터로 받고 있다.

일단 new Promise로 Promise가 생성 되는 직후부터 resolve나 reject 가 호출되기 전까지 순간을 pending 상태라고 봄.

이후 비동기 작업이 마친 뒤 결과물을 약속대로 잘 줄 수 있다면 첫번째 파라미터로 주입되는 resolve 함수를 호출, 실패했다면 reject 함수를 호출한다.

<br/>

## 실행부

_promise()를 호출하면 Promise 객체가 리턴 됨. 정상적으로 비동기 작업이 완료되면 then을 호출. 위의 예제는 `하나의` then API를 호출해 비동기 작업이 완료되면 결과에 따라 성공 혹은 실패 메세지를 로그에 찍어줌. 선언부의 resolve 와 reject 파라미터를 받았는데 그 파라미터가 실행부의 함수와 동일하다. 

<br/>

## Promise.catch

만약 체이닝 형태로 연결된 상태에서 비동기 작업이 중간에 에러가 난다면? catch는 `.then(null, function(){})` 을 메서드 형태로 바꾼 것이라고 생각해도 좋음.

```javascript
_promise(true)
  .then(JSON.parse)
  .catch(function() {
    window.alert('체이닝 중간에 에러남!');
  })
  then(function(text) {
    console.log(text);
  });
```

앞서 _promise 에서 만든 객체는 성공 혹은 실패 시 JSON 객체가 아닌 String을 리턴하므로 JSON.parse 에서 Error가 나게 됨. 따라서 다음 then으로 이동하지 못하고 catch에서 받게 됨. catch는 이와 같이 promise가 연결되어 있을 때 발생하는 오류를 처리해주는 역할을 함.

<br/>

## Promise.all

여러 개의 비동기 작업들이 존재하고 이들 모두 완료되었을 때 작업을 진행하고 싶다면, all 사용. 

여기서 눈여겨 봐야할 것은 promise1,2에 할당된 녀석들은 return 이 붙지 않았다. 바로 new를 통해 객체를 생성해줌. 그렇다면 각자가 .thne을 이어 사용할 수 없다는 말임.

```javascript
var promise1 = new Promise(function (resolve, reject) {
  
  // 비동기를 표현하기 위해 setTimeout 함수 사용
  setTimeout(function() {
    
    // 해결 됨
    console.log('첫번째 Promise 완료');
    resolve('1111');
  }, Math.random() * 20000 + 1000);
});

var promise2 = new Promise(function (resolve, reject) {
  
  // 비동기를 표현하기 위해 setTimeout 함수 사용
  setTimeout(function() {
    
    // 해결 됨
    console.log('두번째 Promise 완료');
    resolve('2222');
  }, Math.random() * 10000 + 1000);
}

Promise.all([promise1, promise2]).then(function (values) {
  console.log('모두 완료됨', values);
});
```

두번째 Promise가 완료된 뒤, 시간이 흘러 첫번째 Promise가 완료되면 최종적으로 전체값을 보여줌.

<br/>



## return 하지 않고 바로 new Promise로 생성하기

항상 new Promise를 return 하는 형태로 사용하다 바로 위의 Promise.all 에 대해 설명할 때는 return이 아닌 바로 new Promise를 할당하는 형태로 사용했다. 어떤 차이가 있나?

```javascript
var _promise = new Promise(function(resolve, reject) {
  
  // 여기서 무언가 수행
  
  // 50% 확률로 resolve
  if (+new Date() % 2 === 0) {
    resolve("Stuff worked!");
  } else {
    reject(Error("It broke"));
  }
});
```

위와 같이 선언한다면 Promise 객체에 파라미터로 넘겨준 익명함수는 즉시 실행된다. 즉각 실행되므로 _promise.then(alert) 등의 형태로 사용할 수 없다.

이후 여러 차례 _promise.then(alert)을 호출해도 이미 한번 수행 되었기 때문에 계속해서 resolve혹은 reject가수행 될 것이다. _promise.then(alert).catch(alert)을 여러 차례 수행해보면, "Stuff wored!" 가 나왔다면, 몇 번을 반복해서 수행해도 계속 "Stuff worked!"가 나오게 됨.

Promise 객체를 new 로 바로 생성할 경우 아래와 같은 형태로 사용 가능하다.

```javascript
var _promise = new Promise(function(resolve, reject) {
  
  // 50% 확률로 resolve
  if (+new Date() % 2 === 0) {
    resolve("Stuff worked!");
  } else {
    reject(Error("It broke"));
  }
}).then(alert).catch(alert);  // 여기 바로 이어줘야 함.
```

이번에는 Promise.all  에 대한 예제를 Promise를 return 하는 형태로 바꿀 경우 어떻게 변하는지 확인해보면,

```javascript
var promise1 = function() {
  return new Promise(function (resolve, reject) {  // 여기 return 붙었지?
    setTimeout(function() {
      console.log('첫번째 Promise 완료');
      resolve('1111');
    }, Math.random() * 20000 + 1000);
  });
};

var promise2 = function() {
  return new Promise(function (resolve, reject) {  // 여기 return 붙었지?
    setTimeout(function() {
      console.log('두번째 Promise 완료');
      resolve('2222');
    }, Math.random() * 10000 + 1000);
  });
};
```

이렇게 return이 붙은 Promise 객체는

```javascript
Promise.all([promise1, promise2]).then(function (values) {
  console.log('모두 완료됨', values);
});
```

이렇게 사용할 수 없다. Promise 객체가 아니기 때문에 오류를 만난다. 따라서 아래와 같이 실행해야 Promise.all을 사용할 수 있다.

```javascript
Promise.all([promise1(), promise2()]).then(function (values) {   // 괄호 붙였지? 즉시 실행 해줘라.
  console.log('모두 완료됨', values);
});
```

<br/>

> `정리`
>
> 비동기 작업에 거의 필수적이라고 볼 수 있는 Promsie. 항상 return new Promise만 사용하다가 return이 붙은 Promise와 함수에 할당된 Promise 객체의 차이점을 알게 되었다.
>
> Promise.all을 사용하려면 all 안에는 promise 객체를 실행하는 함수 자체가 들어가야함을 배웠다. 
>
> [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise) 에 보면 Promise.all에 대한 설명이 조금 더 나와있는데,
>
> Promise.all(iterable) 이라고 함. `iterable` 내의 모든 프로미스가 이행한 뒤, 어떤 프로미스가 거부하면 즉시 거부하는 프로미스를 반환함. 반환된 프로미스가 이행하는 경우 `iterable` 내의 프로미스가 결정한 값을 모은 배열이 이행 값이다. 반환된 프로미스가 거부하는 경우 `iterable` 내의 거부한 프로미스의 이유를 그대로 사용함. 이 메서드는 여러 프로미스의 결과를 모을 때 유용.