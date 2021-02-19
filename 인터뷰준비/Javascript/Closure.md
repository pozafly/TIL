# Closure

> 출처 : https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#closure

Closure(클로저)는 **두 개의 함수로 만들어진 환경**으로 이루어진 특별한 객체의 한 종류이다. 여기서 **환경**이라 함은 클로저가 생성될 때 그 **범위**에 있던 여러 지역 변수들이 포함된 `context` 를 말한다. 이 클로저를 통해 자바스크립트에는 없는 비공개(private) 속성/메소드, 공개 속성/메소드를 구현할 수 있는 방안을 마련할 수 있다.

## 클로저 생성하기

다음은 클로저가 생성되는 조건임.

1. 내부 함수가 익명 함수로 되어 외부 함수의 반환값으로 사용된다.
2. 내부 함수는 외부 함수의 실행 환경(execution environment)에서 실행된다.
3. 내부 함수에서 사용되는 변수 x는 외부 함수의 변수 스코프에 있다.

```javascript
function outer() {
  var name = `closure`;
  function inner() {
    console.log(name);
  }
  inner();
}
outer();

// closure
```

`outer` 함수를 실행시키는 `context` 에는 `name` 이라는 변수가 존재하지 않는다는 것을 확인할 수 있다. 비슷한 맥락에서 코드를 조금 변경해볼 수 있다.

```javascript
var name = `Warning`;
function outer() {
  var name = `closure`;   // free variable(자유변수)
  return function inner() {
    console.log(name);
  };
}

var callFunc = outer();
callFunc();

// closure
```

- 위 코드에서 `callFunc`를 클로저라고 함.
-  `callFunc` 호출에 의해 `name` 이라는 값이 console에 찍히는데, 찍히는 값은 `Warning` 이 아니라 `closure` 라는 값이다. 
- 즉, `outer` 함수의 context 에 속해있는 변수를 참조하는 것이다. 
- 여기서 `outer` 함수의 지역변수로 존재하는 `name` 변수를 `free variable(자유변수)` 라고 함.

이처럼 외부 함수 호출이 종료되더라도 외부 함수의 지역 변수 및 변수 스코프 객체의 체인 관계를 유지할 수 있는 구조를 클로저라고 함. 보다 정확히는 외부 함수에 의해 반환되는 내부 함수를 가리키는 말.





https://velog.io/@open_h/closure-and-scope 이거 보고 정리하자.