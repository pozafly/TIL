# Scope

> 출처: https://tutorialpost.apptilus.com/code/posts/js/js19-scope/

**밖에서는 안이 보이지 않고 안에서만 밖이 보인다.**

## 실행 컨텍스트(Execution context)

자바스크립트 해석기는 실행 컨텍스트라는 개념을 사용하여 구문을 해석함.

- 실행 컨텍스트: 코드가 실제로 실행되는 영역
- 실행 컨텍스트에는 `전역 컨텍스트`와, 각 함수가 생성한 새로운 실행 컨텍스트인 `함수 컨텍스트`로 나뉨.
  - 전역 컨텍스트: 함수가 아닌 영역에 선언된 코드가 실행되는 컨텍스트.
  - 함수 컨텍스트: 함수 내에서 실행되는 코드들은 각자의 함수 컨텍스트를 가짐.
- 이 각각의 컨텍스트에 의해 변수의 스코프가 결정 됨.

<br/>

## 스코프(Scope)

스코프란, 프로그램의 실행 컨텍스트(Execution Context)에서 보거나 접근할 수 있는 식별자를 말함.

- 전역 스코프 - 변수가 함수 바깥에 선언되어 있거나, 변수 선언 시 var 키워드를 사용하지 않으면 변수는 전역 스코프에 포함 됨.
- 함수 스코프 - 변수가 함수 내에서 선언되면 이 변수는 함수 스코프 내에 존재하게 됨.

자바스크립트 변수는 변수가 접근할 수 있는 범위(스코프)에 따라 **전역변수**와 **지역변수**로 나눌 수 있다.

- 전역변수 - 전역 스코프, 즉 함수 바깥에서 선언된 변수임. 전역변수는 프로그램의 전체에서 접근 가능함.
- 지역변수 - 함수 스코프, 즉 함수 내부에 선언된 변수로, 함수 내부에서만 접근 가능함.

```javascript
var global = 1;

function localFunc() {
  var local = 2;
  console.log(global);  // 1
  console.log(local);   // 2
}

localFunc();
console.log(global);  // 1
console.log(local)    // ReferenceError: local is not defined
```

위 코드와 같이 전역변수로 선언한 `global` 변수의 경우 함수 내부 혹은 전역에서 모두 사용 가능함. 하지만 함수 내부에 선언한 `local` 변수의 경우 전역에서 사용하려고 하는 경우 정의되지 않았다고 오류를 뱉음.

<br/>

이렇게 변수의 유효범위를 두는 이유?

- 변수명의 충돌을 방지하기 위해서.
- `local` 이라는 이름의 변수는 `localFunc` 라는 함수 내부에서만 유효하므로, 다른 함수에서도 동일한 변수 명을 사용할 수 있게 됨.
- 하지만 `global` 이라는 변수 명은 다른 곳에서 사용할 경우 문제를 일으킬 가능성이 높아지게 됨.

<br/>

## 스코프 체인

만약 위 예제에서 `localFunc()` 함수 내부에도 `global` 변수를 선언하면 어떻게 됨? 당연히 함수 내부에서 선언한 변수를 사용하게 되고 함수 내부에 `global` 변수가 존재하므로 굳이 전역까지 올라가서 변수를 찾지 않음.

범위를 계속 넓혀가며 결국 전역 스코프까지 찾는 것을 `스코프 체인` 이라고 함.

<hr/>

<br/>

## ES6에서 스코프

> 출처: https://velog.io/@open_h/closure-and-scope

### 함수 레벨 스코프와 블록 레벨 스코프

ES6에서는 **함수 레벨**과 **블록 레벨**(ES6)의 **렉시컬 스코프**(Lexical Scope) 규칙을 따른다.

`var` 로 선언된 변수는 함수 레벨 스코프를 가졌기에, `let` 과 `const` 를 지원하는 ES6가 되어서야 자바스크립트에서 블록 레벨 스코프를 지원하게 되었다. ES6를 사용하는 요즘은 **혼란을 야기하는 함수 레벨 스코프**를 가지는 `var` 대신 블록 레벨 스코프를 사용하는 `let` 과 `const` 로 모두 대체하고 있다. 전부 대체 가능.

```javascript
function f() {
  if(true) {
    var a = 10;  // function level scope
  }
  console.log(a)   // 10
}

function g() {
  if(true) {
    let a = 10;   // block level scope
  }
  console.log(a)  // ReferenceError: a is not defined
}
```

<br/>

### 렉시컬 스코프(Lexical Scope)

렉시컬 스코프는 함수를 어디서 호출하는지가 아니라 **어디에 선언하였는지**에 따라 결정됨.

렉시컬 스코프는 보통 동적 스코프(Dynamic Scope)와 비교하며 자바스크립트는 렉시컬 스코프 규칙을 사용한다. 현대 프로그래밍에서 언어들은 대부분 렉시컬 스코프 규칙을 따르고 있다. 렉시컬 스코프에 대해 설명하기 전, 먼저 예시 코드를 살펴보자.

```javascript
let a = 'a in global';

function foo() {
  let a = 'a in foo';
  bar();
}
function bar() {
  console.log(a);
}

bar();   // a in global
foo();   // a in global
```

만약 `foo()` 에서 `a` 를 선언한 것이 아니라 그저 재할당 하여 전역 변수를 바꾸었다면 위 코드와 콘솔 결과는 달라짐. 하지만 위와 같은 코드라면 두 함수 실행 결과가 'a in global'로 나타남. 이러한 스코핑 규칙을 **렉시컬 스코프** 라고 부름.

스코프는 함수를 호출할 때가 아니라 **선언** 할 때 생기는 것. 레시컬 스코프는 함수를 처음 **선언하는 순간** 함수 내부의 변수는 자기 **스코프로부터 가장 가까운** 곳(스코프 체인)에 있는 변수를 계속 참조하게 됨. 즉, `bar` 이 선언 되었을 때, 이미 `a` 는 함수 내에 `a` 라는 식별자가 없으니 스코프 체인으로 글로벌 변수 `a` 를 참조하게 된다.

<br/>

### 동적 스코프(Dynamic Scope)

위 코드가 렉시컬 스코프가 아니라 *동적 스코프* 규칙을 따른다고 가정하면 아래와 같은 결과가 나옴.

```javascript
bar();   // a in foo
foo();   // a in global
```

렉시컬 스코프와 달리 동적 스코프는 호출 스택(Call Stack)에 영향을 받으며 위 코드대로라면 가장 아래에는 `foo` 함수를 호출할 때, `Global` 스택이 쌓여있는 상태에서, 그 위에 `foo`가 쌓이고 그 다음에는 `bar` 함수가 스택으로 쌓인다. 동적 스코프는 이 stack의 영향을 받기 때문에 `bar` 에서는 `a` 에 접근할 때, 그 아래 stack인 `foo` 의 `a` 에 접근하게 되어 `foo()` 는 'a in global'이 아닌 'a in foo'를 출력하게 됨.

렉시컬 스코프와 마찬가지로 `bar()` 를 호출하였을 때는 바로 'a in global'이 출력되는데 이는 `Global` 스택 위 `bar` 스택이 전부이기에 `Global` 다음 스택의 전역 변수 `a`에 접근하기 때문임. 실제로 동적 스코프 방식은 Perl 과 같은 언어에서 나타는 결과임.

![스크린샷 2021-02-20 오전 10 39 31](https://user-images.githubusercontent.com/59427983/108578932-f722a280-7367-11eb-91e9-fb0d2137139b.png)

<br/>

### 자바스크립트와 렉시컬 스코프

자바스크립트는 렉시컬 스코프 규칙을 따르기 때문에 **호출 스택과 상관 없이** (`this`제외) "이름: 값" 으로 된 '대응표'를 소스코드를 정의하기 때문에 위 코드는 모두 'a in global'로 나타나는 것이다. 자바스크립트의 스코프는 ECMAScript 언어 명세서에서 **렉시컬 환경**(Lexical environment)와 **환경 레코드**(Environment record)라는 개념으로 정의됨. 여기서 "이름: 값" 이라는 대응표는 환경 레코드라고 볼 수 있고, 렉시컬 환경은 이 환경 레코드와 **상위 렉시컬 환경**(Outer Lexical Environment)에 대한 참조로 이루어짐.

현재 환경 레코드에서 변수를 찾아보고 없다면 바깥 렉시컬 환경을 참조하는 식의 스코프 체인 방식이 가능해지며 이름을 찾는데 성공하거나 바깥 렉시컬 환경 참조가 `null` 이 되었을 때 (실패했을 때) 탐색을 멈춘다.

![스크린샷 2021-02-20 오전 10 44 20](https://user-images.githubusercontent.com/59427983/108579062-9a73b780-7368-11eb-862e-4954c9375d43.png)

> `정리`
>
> 스코프는 변수의 적용 범위가 어디까지인지 구분지어주는 범위의 단위. 따라서 범위가 지정되면 유효 범위 안에서 객체 의미를 부여할 수 있다. 렉시컬 스코프를 이해하면 클로저라 불릴 수 있는 함수를 정의할 수 있다.
