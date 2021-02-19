# 함수표현식 vs 함수선언식

> 출처 : https://joshua1988.github.io/web-development/javascript/function-expressions-vs-declarations/

## 함수 선언식 - Function Declarations

일반적인 프로그래밍 언어에서의 함수 선언과 비슷.

```javascript
function 함수명() {
  구현 로직
}
```

<br/>

## 함수 표현식 - Function Expressions

유연한 자바스크립트 언어의 특징을 활용한 선언 방식

```javascript
var 함수명 = function() {
  구현 로직
}
```

<br/>

## 함수 선언식과 표현식의 차이점

함수 선언식은 호이스팅에 영향을 받지만, 함수 표현식은 호이스팅에 영향을 받지 않는다. 

```javascript
logMessage();
sumNumbers();

function logMessage() {
  return 'worked';
}

var sumNumbers = function() {
  return 10 + 20;
}
```

호이스팅에 의해 자바스크립트 해석기는 코드를 아래와 같이 인식한다.

```javascript
// 실행 시
function logMessage() {
  return 'worked';
}

var sumNumbers;

logMessage();  // 'worked'
sumNumbers();  // Uncaught TypeError: sumNumbers is not a function

sumNumbers = function() {
  return 10 + 20;
};
```

함수 표현식 sumNumbers에서 var도 호이스팅이 적용되어 위치가 상단으로 끌어올려졌음.

하지만 실제 sumNumbers에 할당될 function 로직은 호출된 이후에 선언되므로, sumNumbers는 함수로 인식하지 않고 변수로 인식한다.

> 호이스팅을 제대로 모르더라도 함수와 변수를 가급적 코드 상단부에 선언하면, 호이스팅으로 인한 스코프 꼬임 현상은 방지할 수 있음.

<br/>

## 함수 표현식의 장점

**함수 표현식이 호이스팅에 영향을 받지 않는다**는 특징 이외에도 함수 선언식보다 유용하게 쓰이는 경우는 다음과 같음.

- 클로저로 사용
- 콜백으로 사용(다른 함수의 인자로 넘길 수 있음)

<br/>

## 함수 표현식으로 클로저 생성하기

