# Fizzbuzz

다음을 return하는 function을 작성해라.

1. 매개변수는 n인 정수.
2. 3의 배수이면 'Fizz'.
3. 5의 배수이면 'Buzz'.
4. 3과 5의 배수이면 'FizzBuzz'.

나의 답

```js
function fizzBuzz(n) {
  if (n % 3 === 1 && n % 5 === 1) {
    return 'FizzBuzz';
  } else {
    if (n % 3 === 1) return 'Fizz';
    if (n % 5 === 1) return 'Buzz';
    return;
  }
}
```

정답

```js
function fizzBuzz(n) {
  if(n % 3 === 0) return 'Fizz';
  if(n % 5 === 0) return 'Buzz';
  if(n % 3 === 0 && n % 5 === 0) return 'FizzBuzz';
}
```

<br/>

## 풀이

먼저 % 기호는 나머지인데, 1이 나머지가 남는다는 뜻이므로 0을 넣어주어야 한다. 그러면 3배수, 5배수가 되겠지?

그리고 `n % 3 === 1 && n % 5 === 1` 이 구문은 연산이 한번 들어가고 밑에 녀석들을 체크하는 것이기 때문에 나중에 적어주는 것이 효율적이다. 따라서 복잡한 구문은 제일 마지막에 두어 연산을 최소화 하는 것이 좋다. 이 부분은 o(n) 복잡도? 이 부분과 연관이 있다. 시간 복잡도에 대해서 공부해야 함.

제일 마지막에 `return;` 구문은 필요없다.
