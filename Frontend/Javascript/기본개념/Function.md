# Function

> 출처: https://youtu.be/e_lU39U-5bQ

<br/>

## Default parameters (added in ES6)

```javascript
function showMessage(message, from) {
  console.log(`${message} by ${from}`);
}
showMessage('Hi!');

//  Hi! by undefined
```

- from이 전달되지 않았기 때문에 그 부분만 undefined가 뜸.

```javascript
function showMessage(message, from) {
  if (from === undefined) {
    from = 'unknown';
  }
  console.log(`${message} by ${from}`);
}
showMessage('Hi!');

//  Hi! by unknown
```

- 그래서 이전에는 이렇게 if 문을 추가해주어 표현했다. 요새는 잘 쓰지 않음.

```javascript
function showMessage(message, from = 'unknown') {
  console.log(`${message} by ${from}`);
}
showMessage('Hi!');

//  Hi! by unknown
```

- 이렇게 두번째 파라미터에 undefined 시 default 값으로 넣어주면 알아서 처리됨.

<br/>

## Rest parameters (added in ES6)

```javascript
function printAll(...args) {   // ... 은 배열 형태로 전달되어짐
  for (let i = 0; i < args.length; i++) {
    console.log(args[i]);
  }
  for (const arg of args) {
    console.log(arg);
  }
  args.forEach((arg) => console.log(arg));
}
printAll('dream', 'coding', 'ellie');

// 모두 아래와 같은 값이 출력됨.
// dream
// coding
// ellie
```

<br/>

## Early return, Early exit (ES6 아님)

```javascript
// 안좋은 로직
function upgradeUser(user) {
  if (user.point > 10) {
    // long upgrade logic...
  }
}

// 좋은 로직
function upgradeUser(user) {
  if (user.point <= 10) {
    return;
  }
  // long upgrade logic...
}
```

- 블럭 안에서 로직을 작성하게 되면 길어질 수 있고 가독성이 좋지 않음.
- 점철된 if else 보다는 `조건이 맞지 않을 때` **빨리 함수를 종료**하고 조건이 맞을 때만 필요한 로직을 작성해 주면된다.

<br/>

## Callback Function (ES6 아님)

```javascript
function randomQuiz(answer, printYes, printNo) {
  if (answer === 'love you') {
    printYes();
  } else {
    printNo();
  }
}

const printYes = function() {  // anonymous function
  console.log('yes!');
};
const printNo = function print() {  // named function. 나중에 debugging 할 때 유용함. 재귀로 사용할 때도 사용됨.
  console.log('no!');
};

randomQuiz('wrong', printYes, printNo);  // no!
randomQuiz('love you', printYes, printNo);  // yes!
```

- 2, 3번째 매개변수로 function을 넘겼다. 상황에 맞을 때 이 함수를 불러! 하는 것이 Callback function 임.

<br/>

## Arrow Function (added in ES6)

기본적으로 anonymous(익명) function 임.

```javascript
const simplePrint = function() {
  console.log('simple');
}
const simplePrint = () => console.log('simple');
```

<br/>

## IIFE (Immediately Invoked Function Expression) - 즉시 실행 함수

```javascript
(function hello() {
  console.log('IIFE');
})();
```

바로 실행됨.

> `정리`
>
> 그냥 별 생각 없이 썼던 녀석들을 자세하게 알 수 있어서 좋음. Default parameters, Rest parameters는 한번도 사용해보지 않았는데. react 때 Rest parameters 를 사용하지 않을까?
