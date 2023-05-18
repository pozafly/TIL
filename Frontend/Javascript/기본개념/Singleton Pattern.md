# Singleton Pattern

> 출처
>
> - https://heecheolman.tistory.com/40
> - https://techbless.github.io/2020/06/09/ES7-%EC%9D%B4%EC%83%81%EC%9D%98-TypeScript%EC%99%80-JavaScript%EC%97%90%EC%84%9C-Singleton-%ED%8C%A8%ED%84%B4%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%93%B8%EA%B9%8C/

## 싱글톤패턴이란

싱클톤 패턴은 전체 시스템에서 하나의 인스턴스만 존재하도록 보장하는 객체 생성 패턴이다. 동일 클래스로 `new` 를 해도 인스턴스 하나만 존재한다는 것이다.

싱글톤 패턴을 사용하면 고정된 메모리 영역에 인스턴스 하나만 사용하기 때문에 **메모리 낭비를 방지**할 수 있다. 또한, 싱글톤으로 만들어진 인스턴스는 전역이기 때문에 **다른 클래스의 인스턴스들이 데이터를 공유하기 쉬워진다**.

하지만, 싱글톤 인스턴스가 너무 많은 일을 하거나 많은 데이터를 공유한다면, 시스템의 결합도가 높아지기 때문에 주의해야 한다. 결합도가 높아지면 유지보수가 어려워진다.

## 특징

- 객체 자체에는 접근이 불가능해야 함.
- 객체에 대한 접근자(비공개 멤버 : 클로저)를 사용해 실제 객체를 제어할 수 있다.
- 객체는 단 하나만 만들어지며, 해당 객체를 공유함.

## 구현

다음과 같은 객체 리터럴도 싱글톤 패턴

```js
var obj = {
  a: 'foo',
  b: function(){}
}
```

하지만 이렇게 되면 비공개 상태 및 함수를 정의할 수 없음. 그렇기 때문에 클로저를 통해 비공개로 만들어야 함.

### 클로저 사용

```js
const Singleton = (function () {
  // instance 비공개 변수 정의
  let instance;

  // foo 비공개 메서드 정의
  function foo() {
    return {
      // public 메서드 정의
      publicMethod() {
        return 'Hello Singleton Pattern!';
      },
      // public 속성 정의
      publicProp: 'single value',
    };
  }

  // public 메서드인 getInstance를 정의한 객체
  // 이 메서드를 통해 비공개된 메서드와 변수에 접근 가능 (closure)
  return {
    getInstance() {
      if (!instance) {
        instance = foo();
      }
      return instance;
    },
  };
})();

const a = Singleton.getInstance();
const b = Singleton.getInstance();

console.log(a === b); // true

console.log(a.publicMethod()); // Hello Singleton Pattern!
console.log(b.publicMethod()); // Hello Singleton Pattern!
```

foo()의 return 문에서 객체 리터럴로 정의되는 객체가 Singleton Object임. 이 객체는 프로그램 전체에서 하나만 존재하게 된다. `getInstance()`의 public 메서드를 통해 instance 변수의 값을 확인한 후 아직 Singleton Object가 생성되지 않았다고 판단하면, 내부 함수인 `foo()`를 통해 `Singleton Object`를 반환해 instance에 할당한다.

### ES6

```js
let instance;
class Singleton {
  constructor() {
    if (instance) return instance;
    this.name = 'hst';
    this.age = 10;
    instance = this;
  }
}

const firstSingleton = new Singleton();
const secondSingleton = new Singleton();

console.log(firstSingleton === secondSingleton); // true
console.log(instance === secondSingleton); // true
```

instance의 캡슐화가 되지 않아 완벽하진 않다.

### ES7

```js
class Singleton {
  static instance;

  constructor() {
    if (Singleton.instance) return Singleton.instance;
    this.name = 'hst';
    this.age = 10;
    Singleton.instance = this;
  }
}

const firstSingleton = new Singleton();
const secondSingleton = new Singleton();

console.log(firstSingleton === secondSingleton); // true
console.log(Singleton.instance === secondSingleton); // true
```































