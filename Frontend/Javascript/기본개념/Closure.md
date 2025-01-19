# Closure

> 출처: https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#closure

Closure(클로저)는 **두 개의 함수로 만들어진 환경**으로 이루어진 특별한 객체의 한 종류이다. 여기서 **환경**이라 함은 클로저가 생성될 때 그 **범위**에 있던 여러 지역 변수들이 포함된 `context` 를 말한다. 이 클로저를 통해 자바스크립트에는 없는 비공개(private) 속성/메소드, 공개 속성/메소드를 구현할 수 있는 방안을 마련할 수 있다.

<br/>

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
- `callFunc` 호출에 의해 `name` 이라는 값이 console에 찍히는데, 찍히는 값은 `Warning` 이 아니라 `closure` 라는 값이다.
- 즉, `outer` 함수의 context 에 속해있는 변수를 참조하는 것이다.
- 여기서 `outer` 함수의 지역변수로 존재하는 `name` 변수를 `free variable(자유변수)` 라고 함.

이처럼 외부 함수 호출이 종료되더라도 외부 함수의 지역 변수 및 변수 스코프 객체의 체인 관계를 유지할 수 있는 구조를 클로저라고 함. 보다 정확히는 외부 함수에 의해 반환되는 내부 함수를 가리키는 말.

<br/>

더 자세히 보자.

> 출처: https://velog.io/@open_h/closure-and-scope

클로저란, **함수와 그 함수가 선언되었을 때의 렉시컬 환경(Lexical Environment)과의 조합이다.**

정의상 개념적으로는 모든 함수를 클로저로 칭하는 것 같다. 하지만 실제로 클로저라는 단어를 사용할 때 자바스크립트에서는 **모든 함수를 전부 클로저라 부르지 않는다.** 위 클로저 정의에서 자바스크립트의 경우

- 함수: 반환된 내부함수를 의미
- 렉시컬 환경: 내부 함수가 선언되었을 때의 스코프를 뜻한다.

```javascript
function foo() {
  let a = 10;
  function bar() {
    console.log(a);
  }
  bar();
}
foo();
```

`bar` 는 일단 `foo` 의 안에 있어 `foo`를 상위 환경 참조로 저장하고, 렉시컬 스코프 체인을 통해 한 단계 범위를 넓히면 `foo` 의 `a` 를 참조하게 됨. 마치 클로저처럼 *보이지만* **클로저라고 부르지 않는다.** 다음 예제 코드를 보자.

```javascript
let a = 'apple';
function foo() {
  let a = 'grape';
  function bar() {
    console.log(a);   // 순서 1
  }
  return bar;
}
let yoo = foo();   // 순서 3
yoo();
```

코드의 실행 순서를 `bar` 관점에서 차례로 보자.

1. `bar` 는 `a` 를 출력하는 함수로 정의됨.
2. `bar` 는 상위 환경 참조로 `foo` 의 환경을 저장하였다.
3. `bar` 를 전역 변수 `yoo` 로(함수 선언식으로) 가져왔다.
   - 이 의미는 foo() 를 실행하면서, yoo라는 변수에, bar() 함수를 리턴하면서 foo() 함수가 종료됨을 말한다.
4. 전역에서 `yoo` 를 호출했다.
5. 이제 `bar` 는 자신의 스코프에서 `a` 를 찾지만 없다.
6. 따라서 외부 환경 참조를 찾아간다.
7. 외부 환경인 `foo` 의 스코프에서 `a` 를 찾을 수 있으며 그 값은 'grape' 다.
8. 따라서 'grape' 가 출력된다.

두 번째 예제코드는 첫 번째 예제 코드와 달리 **클로저를 나타내고 있다.** 두 예제 코드는 비슷하지만 클로저를 나타내는데 있어 중요한 차이점이 있음.

1. `bar` 는 **자신이 생성된 렉시컬 스코프에서 벗어나** 전역에서 `yoo` 라는 식별자로 호출되었다.
2. 스코프 탐색은 현재 실행 스택(`yoo` 호출)과 **관련 없는**(동적 스코프와의 차이점임) `foo` 를 거쳤다.
   - 즉, 다시 말하면, foo() 함수는 **종료 되었음에도 불구하고** foo() 함수 안에 있는 `let a = 'grape';` 를 참조했다는 것이다.
   - [생활코딩 클로저 영상](https://www.youtube.com/watch?v=bNqOHnsJm-w) 여길 보자.

다시 말하면, 렉시컬 스코프는 소스코드가 작성된 그 문맥에서 스코프가 결정되는 것임. **이미 `bar` 의 상위 렉시컬 환경을 결정**한 이후에는 `bar` 와 상관없는 곳에서 호출한다 하더라도 `foo` 에서 `a` 를 탐색하게 된다. 이 때, `bar` 또는 `yoo` 와 같은 함수를 클로저라고 부른다.

<br/>

## 클로저의 실용적인 활용

클로저는 자신이 생성될 때의 환경을 기억해야 하므로 클로저가 필요하지 않은 작업에서 함수 내에서 함수를 불필요하게 작성하는 것은 처리 속도와 메모리 측면에서 현명하지 않다고 말한다. 여기서는 클로저가 실용적으로 잘 사용될 수 있는 예시들을 정리함.

<br/>

### 1. 전역 변수의 사용 억제

**전역 변수를 사용하지 않고**도 현재 상태(혹은 값)를 기억하고 변경된 최신 상태를 유지할 수 있다. 어떤 상태를 관리할 때, 클로저가 없다면 (불가피하게) 전역 변수를 사용하거나 아예 로컬스토리지를 사용할 수도 있다. 하지만 클로저를 활용하면 전역 변수가 아니라 타이트한 변수 스코핑을 통해 현명하게 상태나 값을 관리할 수 있다.

박스의 색을 토글하는 함수를 전역 변수를 사용하지 않고 클로저와 즉시실행함수를 활용하여 구현했다. 아래 코드에서 즉시실행함수가 반환한(이름없는) 함수는 렉시컬 환경에 속한 변수를 기억하는 클로저가 된다.

```html
<!DOCTYPE html>
<html>
<body>
  <div class="box" style="width: 100px; height: 100px; background: green;"></div>

  <script>
    // Box Color Toggler
    const box = document.querySelector('.box');

    const toggleColor = (function () {
      let isGreen = true;
      // 클로저 반환
      return function () {
        box.style.background = isGreen ? 'red' : 'green';
        // 상태 변경
        isGreen = !isGreen;
      };
    })();
    // 박스 클릭 이벤트
    box.addEventListener('click', toggleColor);
  </script>
</body>
</html>
```

예시 코드처럼 클로저를 사용할 경우 전역 변수를 사용하지 않게 된다. 단순 `true`, `false` 토글 외에도 클로저가 포함된 함수의 변수를 private 한 변수인 것처럼 쓸 수 있다. 이렇게 불필요한 전역 변수를 사용하지 않음으로서 얻는 이득은 상당함. **의도되지 않은 변경**은 개발자가 걱정할 필요가 없기 때문에 코드가 안정적이게 된다. 필요한 곳에서만 해당 변수에 접근할 수 있게 하고 그 외부에서는 접근하지 못하게 막는 코드는 예상치 못한 부작용을 막을 수 있는 좋은 코드라고 할 수 있다.

<br/>

### 2. 프라이빗 메소드(Private Method) 흉내내기

[MDN link](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures)

다른 몇몇 언어들은 프라이빗으로 선언할 수 있는 기능을 제공하여 같은 클래스 **내부의 다른 메소드에서만** 그 메소드들을 호출할 수 있게 만들어준다. 프라이빗 메소드는 name space를 관리할 수 있게 하고 코드에 대한 접근을 제한적으로 하여 타이트한 스코프를 설계할 수 있게 해준다는 이점이 있다. 자바스크립트에 클래스와 메소드는 있지만, 자체적인 프라이빗 메소드는 따로 없다. 그러나 클로저를 이용하여 **마치 프라이빗 메소드인 것처럼** 구현할 수 있다.

1. 위에 있었던 toggle 예시와 같은 방식으로는 하나의 함수밖에 클로저로 리턴할 수 없어 구현에 제한이 많음. 하지만 아래 예시 코드와 같이 구현할 경우 다양한 함수를 리턴하여 많은 것을 구현할 수 있음. 코드는 프라이빗 함수와 변수에 접근하는 퍼블릭 함수를 클로저를 이용해 구현한 코드이고, 이렇게 클로저를 사용하는 것을 **모듈 패턴**(Module Pattern)이라고 함.

```javascript
let counter = (function() {
  let privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  };
})();

console.log(counter.value());   // 0
counter.increment();
counter.increment();
console.log(counter.value());   // 2
counter.decrement();
console.log(counter.value());   // 1

```

`counter` 라는 객체에 private method인 `increment`, `decrement`, `value` 를 사용하는 것처럼 구현되었다. `counter` 내부의 `privateCounter` 라는 변수에는 개발자가 정해준 (가짜)프라이빗 메소드로만 접근할 수 있는 것임.

1. 아래와 같은 생성자 함수로도 위 MDN의 예시 코드와 같은 목적과 기능을 가진 코드를 구현할 수 있다.

```javascript
function Counter() {
  let counter = 0;
  this.increase = function() {
    return ++counter;
  };
  this.decrease = function() {
    return --counter;
  };
}
const counter = new Counter();

console.log(counter.increase())   // 1
console.log(counter.increase())   // 2
console.log(counter.increase())   // 3
console.log(counter.decrease())   // 2
```

<br/>

> `정리`
>
> 클로저의 활용을 보고 이제 이해가 간다. 클로저라고 칭할 때는 한 함수의 내부 함수를 전역에서 부를 수 있는지 없는지에 따라 나뉘는 것 같다.
>
> 클로저의 목적은 전역변수가 많아지면 관리하기 어렵고 변수명이 겹칠 수 있으며 이를 의미 단위의 함수로 쪼개어 사용할 수 있게 하는 듯. 그리고, 마찬가지로 클로저 안에 선언된 메서드를 private 처럼 만들어 전역 단위에서 사용하게끔 함,
>
> 그냥 별 생각없이 기능 구현에만 초점을 두면 전역변수를 막 사용해서 나만 알 수 있는 코드가 되는 반면 클로저를 활용하면 기능 단위 즉 오브젝트 별 기능을 제한해둘 수 있어 효율적이고 의미 파악이 쉽겠다.
>
> [생활코딩 클로저 응용](https://www.youtube.com/watch?v=9A0pMrS6Bh0) 이부분을 보자.
